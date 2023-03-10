<script>
import UmlWebClient from 'uml-js/lib/umlClient';
import ElementType from './specComponents/ElementType.vue';
import StringData from './specComponents/StringData.vue';
import SetData from './specComponents/SetData.vue';
import getImage from '../GetUmlImage.vue';
import SingletonData from './specComponents/SingletonData.vue';
export default {
    props: ['umlID'],
    emits: ['specification', 'dataChange'],
    inject: ['dataChange'],
    data() {
        return {
            elementType: '',
            elementImage: undefined,
            elementData: {
                ownedElements : [],
                owner : undefined,
                appliedStereotypes: [],
                // etc...
            },
            namedElementData : undefined,
            isFetching: true
        }
    },
    mounted() {
        this.reloadSpec();
    },
    watch: {
        umlID(newID, oldID) {
            this.reloadSpec();
        },
        dataChange(newDataChange, oldDataChange) {
            if (newDataChange.id === this.umlID && newDataChange.type === 'name') {
                this.namedElementData.name = newDataChange.value;
            }
        }
    },
    methods: {
        async reloadSpec() {
            this.isFetching = true;
            const client = new UmlWebClient(this.$sessionName);
            const el = await client.get(this.umlID);
            this.elementType = el.elementType();
            this.elementImage = getImage(el);
            this.elementData.ownedElements = [];
            for await(let ownedElement of el.ownedElements) {
                this.elementData.ownedElements.push({
                    img: getImage(ownedElement),
                    label: ownedElement.name !== undefined && ownedElement.name !== '' ? ownedElement.name : ownedElement.id,
                    id: ownedElement.id
                })
            }
            const owner = await el.owner.get();
            if (owner !== undefined) {
                this.elementData.owner = {
                    img: getImage(owner),
                    label: owner.name !== undefined && owner.name !== '' ? owner.name : owner.id,
                    id: owner.id
                }
            } else {
                this.elementData.owner = undefined;
            }
            // TODO rest of ELEMENT


            if (el.isSubClassOf('namedElement')) {
                this.namedElementData = {
                    name: el.name
                };
                const namespace = await el.namespace.get();
                if (namespace !== undefined) {
                    this.namedElementData.namespace = {
                        img: getImage(namespace),
                        label: namespace.name !== undefined && namespace.name !== '' ? namespace.name : namespace.id,
                        id: namespace.id
                    }
                }
            } else {
                this.namedElementData = undefined;
            }

            if (el.isSubClassOf('namespace')) {
                this.namespaceData = {};
                this.namespaceData.members = [];
                for await (let member of el.members) {
                    this.namespaceData.members.push({
                        img: getImage(member),
                        label: member.name !== undefined && member.name !== '' ? member.name : member.id,
                        id: member.id
                    });
                }
                this.namespaceData.ownedMembers = [];
                for await (let member of el.ownedMembers) {
                    this.namespaceData.ownedMembers.push({
                        img: getImage(member),
                        label: member.name !== undefined && member.name !== '' ? member.name : member.id,
                        id: member.id
                    });
                }
            } else {
                this.namespaceData = undefined;
            }

            if (el.isSubClassOf('packageableElement')) {
                this.packageableElementData = {};
                const owningPackage = await el.owningPackage.get();
                if (owningPackage !== undefined) {
                    this.packageableElementData.owningPackage = {
                        img: getImage(owningPackage),
                        label: owningPackage.name !== undefined && owningPackage.name !== '' ? owningPackage.name : owningPackage.id,
                        id: owningPackage.id
                    };
                }
            } else {
                this.packageableElementData = undefined;
            }

            if (el.isSubClassOf('package')) {
                this.packageData = {};
                this.packageData.packagedElements = [];
                for await (let packagedElement of el.packagedElements) {
                    this.packageData.packagedElements.push({
                        img: getImage(packagedElement),
                        label: packagedElement.name !== undefined && packagedElement.name !== '' ? packagedElement.name : packagedElement.id,
                        id: packagedElement.id
                    });
                }
            } else {
                this.packageData = undefined;
            }

            this.isFetching = false;
        },
        propogateSpecification(spec) {
            this.$emit('specification', spec);
        },
        propogateDataChange(dataChange) {
            if (dataChange.type === 'name') {
                this.namedElementData.name = dataChange.value;
            }
            this.$emit('dataChange', dataChange);
        }
    },
    computed: {
        elementLabel() {
            if (this.namedElementData !== undefined && this.namedElementData.name !== '') {
                return this.namedElementData.name;
            } else {
                return this.umlID;
            }
        },
    },
    components: { ElementType, StringData, SetData, SingletonData }
}
</script>
<template>
    <div class="mainDiv" v-if="!isFetching">
        <div class="headerDiv">
            <h1>
                Specification of {{ elementType }} {{ elementLabel }}
            </h1>
            <img v-bind:src="elementImage" v-if="elementImage !== undefined" class="headerImage"/>
        </div>
        <ElementType :element-type="'Element'">
            <StringData :label="'ID'" :initial-data="umlID" :read-only="true" :umlid="umlID" :type="'id'" @data-change="propogateDataChange"></StringData>
            <SetData :label="'Owned Elements'" :initial-data="elementData.ownedElements" :umlid="umlID" :subsets="['ownedAttributes', 'packagedElements']" 
                    @specification="propogateSpecification"></SetData>
            <SingletonData :label="'Owner'" :inital-data="elementData.owner" :uml-i-d="umlID" @specification="propogateSpecification"></SingletonData>
        </ElementType>
        <ElementType :element-type="'Named Element'" v-if="namedElementData !== undefined">
            <StringData :label="'Name'" :initial-data="namedElementData.name" :read-only="false" :umlid="umlID" :type="'name'" 
            @data-change="propogateDataChange"></StringData>
            <SingletonData :label="'Namespace'" :inital-data="namedElementData.namespace" :uml-i-d="umlID" @specification="propogateSpecification"></SingletonData>
        </ElementType>
        <ElementType :element-type="'Namespace'" v-if="namespaceData !== undefined">
            <SetData :label="'Members'" :initial-data="namespaceData.members" :umlid="umlID" :subsets="['ownedAttributes', 'packagedElements']"
                    @specification="propogateSpecification"></SetData>
            <SetData :label="'Owned Members'" :initial-data="namespaceData.ownedMembers" :umlid="umlID" :subsets="['ownedAttributes', 'packagedElements']"
                        @specification="propogateSpecification"></SetData>
        </ElementType>
        <ElementType :element-type="'Packageable Element'" v-if="packageableElementData !== undefined">
            <SingletonData :label="'OwningPackage'" :inital-data="packageableElementData.owningPackage" :uml-i-d="umlID" @specification="propogateSpecification"></SingletonData>
        </ElementType>
        <ElementType :element-type="'Package'" v-if="packageData !== undefined">
            <SetData :label="'Packaged Elements'" :initial-data="packageData.packagedElements" :umlid="umlID" :subsets="['packagedElements']"
                    @specification="propogateSpecification"></SetData>
        </ElementType>
    </div>
</template>
<style>
.mainDiv {
    border: solid #525258;
    border-width: 2px;
    padding: 10px;
    margin:auto;
    width: 1000px;
    height: 100%;
    overflow: auto;
}
.headerDiv {
    display: flex;
}
.headerImage {
    height: 50px;
    width: 50px;
    padding-left: 10px;
}
</style>
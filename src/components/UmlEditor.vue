<script>
import e from 'cors';
import UmlWebClient from 'uml-js/lib/umlClient';
import CloseSymbol from './icons/close_symbol.svg';
import SpecificationPage from './SpecificationPage.vue';
import WelcomePage from './WelcomePage.vue';
export default {
    props: [
        "tabs",
        "specificationTab"
    ],
    inject: ['dataChange'],
    emits: ['specification', 'dataChange'],
    data() {
        return {
            closeSymbol: CloseSymbol,
            welcome: false,
            specification: false,
            diagram: false,
            recentTab: ''
        };
    },
    mounted() {
        if (this.tabs.length > 0) {
            switch(this.tabs[0].type) {
                case 'Welcome': {
                    this.welcome = true;
                    break;
                }
                case 'Specification': {
                    this.specification = true;
                    break;
                }
                case 'Diagram': {
                    this.diagram = true;
                    break;
                }
            }
        }
    },
    watch: {
        recentTab(newRecentTab, oldRecentTab) {
            this.focus(newRecentTab);
        },
        specificationTab(newRecentTab, oldRecentTab) {
            this.recentTab = newRecentTab;
        },
        dataChange(newDataChange, oldDataChange) {
            if (newDataChange.type === 'name') {
                this.tabs.forEach(tab => {
                    if (tab.id === newDataChange.id) {
                        if (newDataChange.value === '') {
                            tab.label = tab.id;
                        } else {
                            tab.label = newDataChange.value;
                        }
                    }
                });
            }
        }
    },
    methods: {
        remove(id) {
            const tabIndex = this.tabs.findIndex(tab => tab.id === id);
            const tab = this.tabs[tabIndex];
            this.tabs.splice(tabIndex , 1);
            if (tab.isActive) {
                if (tabIndex === 0) {
                    this.welcome = false;
                    this.specification = false;
                    this.diagram = false;
                }
                this.recentTab = this.tabs[tabIndex - 1].id;
            }
        },
        focus(id) {
            for (let tab of this.tabs) {
                if (tab.id === id) {
                    tab.isActive = true;
                    this.welcome = false;
                    this.specification = false;
                    this.diagram = false;
                    if (tab.type === "Welcome") {
                        this.welcome = true;
                    } else if (tab.type === 'Specification') {
                        this.specification = true;
                    } else if (tab.type === 'Diagram') {
                        this.diagram = true;
                    }
                }
                else {
                    tab.isActive = false;
                }
            }
        },
        updateTab(id) {
            this.recentTab = id;
        },
        propogateSpecification(spec) {
            this.$emit('specification', spec);
        },
        propogateDataChange(dataChange) {
            if (dataChange.type === 'name') {
                this.tabs.forEach(tab => {
                    if (tab.id === dataChange.id) {
                        tab.label = dataChange.value;
                    }
                });
            }
            this.$emit('dataChange', dataChange);
        }
    },
    components: { WelcomePage, SpecificationPage }
}
</script>
<template>
    <div class="umlEditor">
        <div class="tabContainer">
            <div v-for="tab in tabs" :key="tab.id" class="tab" :class="tab.isActive ? 'activeTab' : 'tab'" @click="updateTab(tab.id)">
                <img v-bind:src="tab.img" v-if="tab.img !== undefined" class="tabImage"/>
                <div style="float:left;padding:5px;">{{ tab.label }}</div>
                <img v-bind:src="closeSymbol" @click.stop="remove(tab.id)" style="padding:5px"/>
            </div>
        </div>
        <div class="activeEditor">
            <WelcomePage v-if="welcome"></WelcomePage>
            <SpecificationPage v-if="specification" :uml-i-d="recentTab" @specification="propogateSpecification" @data-change="propogateDataChange"></SpecificationPage>
        </div>
    </div>
</template>
<style>
    .umlEditor {
        flex: 1 1 auto;
        display: block;
    }
    .tabContainer {
        float: left;
        width: 100%;
        background-color: var(--vt-c-black);
        min-height: 34px;
    }
    .tab {
        display: flex;
        align-items:center;
        float: left;
        background-color: var(--vt-c-dark-dark);
    }
    .tabImage {
        padding-left: 5px;
    }
    .activeTab {
        vertical-align: middle;
        float: left;
        background-color: #2d3035;
    }
    .activeTab:hover {
        background-color: #383c46;
    }
    .tab:hover {
        background-color: var(--vt-c-dark-soft);
    }
    .activeEditor {
        background-color: #2d3035;
        height: 100%;
        clear:both;
        padding: 10px;
    }
</style>
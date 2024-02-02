<template>
    <div class="list-container">
        <div class="title">
            {{ title }}
        </div>
        <div class="list">
            <template v-for="(value, key) of templateRights" :key="key">
                <div class="label" @mouseenter="mouseEv(key)" @mouseleave="mouseEv(defaultKey)">
                    <div class="checklabel" :class="{'end':value !== Object(value)}">
                        <div>
                            <template v-if="value === Object(value)">
                                <input :id="key" type="checkbox" @change="(ev) => change(ev)"
                                       :checked="rights != undefined && key in rights" />
                            </template>
                            <template v-else>
                                <input :id="key" type="checkbox" @change="(ev) => change(ev)"
                                       :checked="rights != undefined && key in rights && rights[key]==1" />
                            </template>
                            <label :for="key">{{ titlesRights ? titlesRights[key].title : key }}</label>
                        </div>
                        <button v-if="value === Object(value)">
                            <SvgArrowRight :width="24" :height="24" />
                        </button>
                    </div>
                    <template v-if="hoveredKey == key">
                        <template v-if="value === Object(value)">
                            <template v-if="titlesRights != undefined && key in titlesRights">
                                <List :templateRights="templateRights[key]" :titlesRights="titlesRights[key].items"
                                      :rights="rights?rights[key]:undefined" :titles="nextTitles"
                                      :style="{'--offset': Object.keys(templateRights).indexOf(key)}"
                                      @keyChange="stackChange" />
                            </template>
                            <template v-else>
                                <List :templateRights="templateRights[key]" :titlesRights="undefined"
                                      :rights="rights?rights[key]:undefined" :titles="nextTitles"
                                      :style="{'--offset': Object.keys(templateRights).indexOf(key)}"
                                      @keyChange="stackChange" />
                            </template>
                        </template>
                    </template>
                </div>
            </template>
        </div>
    </div>
</template>

<script>
import List from "./List";
import { ref, toRaw } from "vue";
import { cloneObject } from "@/utils/functions";
import SvgArrowRight from "@/components/right-editor/SvgArrowRight";

export default {
    name: "List",
    components: { SvgArrowRight, List },
    props: {
        rights: {
            type: Object
        },
        templateRights: {
            type: Object
        },
        titlesRights: {
            type: Object

        },
        titles: {
            type: Array,
            default: []
        }
    },
    setup(props, context) {
        const defaultKey = "";
        const hoveredKey = ref(defaultKey);

        const localTitles = cloneObject(props.titles);
        const nextTitles = localTitles.splice(1)
        const title = toRaw(props.titles[0]);

        const mouseEv = (key) => {
            hoveredKey.value = key;
        };

        const change = (ev) => {
            context.emit("keyChange", { value: ev.target.checked, key: [hoveredKey.value] });
        };

        const stackChange = (key) => {
            context.emit("keyChange", { value: key.value, key: [hoveredKey.value, ...key.key] });
        };

        return {
            hoveredKey,
            mouseEv, defaultKey,
            change, stackChange,
            localTitles, title, nextTitles
        };
    }
};
</script>

<style lang="scss" scoped>

.list-container {
    height: calc((var(--max-rows) + 1) * 30px + 2px);
    top: calc(-1 * (var(--offset, 0) + 1) * 30px - 1px);
    width: max-content;

    .title {
        display: flex;
        height: 20px;
        align-items: center;
        padding: 5px;
    }

    .list {

        grid-template-rows: repeat(auto-fit, 30px);
        display: grid;
        width: max-content;
        height: calc(var(--max-rows) * 30px);
        border: 1px solid #748797;

        .label {
            display: flex;
            position: relative;

            .checklabel {
                display: flex;
                gap: 5px;
                align-items: center;
                align-content: center;
                justify-content: space-between;
                height: 20px;
                padding: 5px;
                min-width: 150px;

                & > div {
                    display: flex;
                    gap: 5px;
                    cursor: pointer;

                    & > input {
                        cursor: pointer;
                    }

                    & > label {
                        color: #162133;
                        font-family: 'Inter', sans-serif;
                        font-weight: 500;
                        font-size: 14px;
                        line-height: 20px;
                        cursor: pointer;
                    }
                }


                button {
                    cursor: pointer;
                    display: flex;
                    justify-content: center;
                    align-content: center;
                    align-items: center;
                    border: none;
                    background-color: transparent;
                    color: #748797;
                    text-decoration: none;
                }

                &.end {
                    justify-content: start;

                    button {
                        display: none;
                    }
                }
            }

            .list-container {
                display: none;
                position: absolute;
                left: 100%;
            }

            &:hover {
                background-color: #162133;

                & > .checklabel > div > label {
                    color: #FFFFFF;
                }

                & > .list-container {
                    display: block;
                }
            }

        }
    }
}

</style>

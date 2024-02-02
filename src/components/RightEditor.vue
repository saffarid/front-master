<template>
    <div class="editor-permission">
        <div class="list-wrap" @wheel="toScroll" ref="editor" :style="styleVars">
            <List
                :templateRights="permission"
                :titlesRights="titles"
                :titles="['Название', 'Проекты', 'Действия']"
                :rights="perm"
                @keyChange="changePerm" />
        </div>
        <div class="buttons">
            <button @click="save">Сохранить</button>
        </div>
    </div>
</template>

<script>
import {
    onMounted,
    reactive,
    ref, toRaw
} from "vue";
import List from "@/components/List";
import set from "set-value";
import API from "@/api/api";
import { cloneObject } from "@/utils/functions";

export default {
    name: "RightEditor",
    components: { List },

    setup(props) {
        const editor = ref(null);
        const perm = reactive({});
        const initTemplatePerm = {};
        const titles = reactive({});
        const permission = reactive({});
        const styleVars = reactive({
            "--max-rows": 1
        });
        const isObj = (value) => value === Object(value);

        const objectCopy = (copied, target) => {
            for (const key of Object.keys(copied)) {
                if (isObj(copied[key])) {
                    if (!(key in target)) target[key] = {};
                    objectCopy(copied[key], target[key]);
                } else {
                    target[key] = copied[key];
                }
            }
        };

        const changePerm = (ev) => {
            if (ev.value) {
                let templatePerm = cloneObject(initTemplatePerm);
                let p = {};
                for (let key of ev.key) {
                    templatePerm = templatePerm[key];
                }

                if (isObj(templatePerm)) {
                    const handleTemplate = (template) => {
                        for (let key of Object.keys(template)) {
                            if (isObj(template[key])) handleTemplate(template[key]);
                            else template[key] = 1;
                        }
                    };

                    handleTemplate(templatePerm);
                    set(p, ev.key, templatePerm);
                } else set(p, ev.key, 1);
                objectCopy(p, perm);
            } else {
                const del = (obj) => {
                    for (let key of ev.key) {
                        if (!(key in obj)) continue;
                        if (isObj(obj[key])) {
                            del(obj[key]);
                            if (ev.key.indexOf(key) + 1 == ev.key.length) {
                                delete obj[key];
                            } else if (Object.keys(obj[key]).length == 0) {
                                delete obj[key];
                            }
                        } else {
                            delete obj[key];
                        }
                    }
                };
                del(perm);
            }
        };

        const toScroll = (ev) => {
            if (editor.value) editor.value.scrollLeft += ev.deltaY;
        };

        const save = () => {
            const copyPerm = {};
            objectCopy(initTemplatePerm, copyPerm);
            objectCopy(perm, copyPerm);
            API.createPermission(copyPerm);
        };

        onMounted(async () => {
            const data = (await API.getTemplates());
            objectCopy(data.data.rootPermissionTitles, titles);
            objectCopy(data.data.rootPermission, permission);
            objectCopy(data.data.rootPermission, initTemplatePerm);

            let maxRows = Object.values(data.data.rootPermission).length;
            const handleArray = (obj) => {
                maxRows = Math.max(Object.values(obj).length, maxRows);
                for (let value of Object.values(obj)) {
                    if (value != Object(value)) continue;
                    handleArray(value);
                }
            };
            handleArray(data.data.rootPermission);
            styleVars["--max-rows"] = maxRows;
        });

        return {
            changePerm,
            editor,
            toScroll,
            perm,
            save,
            titles,
            permission,
            styleVars
        };
    }
};
</script>

<style lang="scss">
.editor-permission {
    .list-wrap {
        min-width: 100%;
        overflow: auto;
    }

    .buttons {
        display: flex;
        justify-content: end;

        button {
            outline: none;
            border: none;
            width: 110px;
            height: 40px;
            border-radius: 4px;
            padding: 10px 16px;
            background-color: #162133;
            color: #FFFFFF;
            font-family: 'Inter', sans-serif;
            font-size: 14px;
            font-weight: 700;
            line-height: 20px;
        }
    }
}

</style>

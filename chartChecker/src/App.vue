<template>
    <n-card
            style="width: 80%;margin-left: 10%;margin-right: 10%"
            title="HELM应用包检查工具"
            bordered
            :segmented="{
      content: true,
       footer: 'soft'
    }"
    >
        <template #header-extra>
            <n-upload
                    directory="true"
                    directory-dnd="true"
                    :show-file-list="false"
                    multiple
                    :max="30"
                    size="huge"
                    style="width: 600px"
                    v-model:file-list="fileList"
                    @change="handleFileChange"
                    @before-upload="handleBeforeUpload"
            >
                <n-upload-dragger>
                    <div>
                        <n-icon size="30" :depth="3" color="green">
                            <UnarchiveSharp/>
                        </n-icon>
                    </div>
                    <n-text style="font-size: 12px">
                        点击或者拖动Chart包到该区域来上传
                    </n-text>
                </n-upload-dragger>
            </n-upload>
        </template>

        <n-space>
            <n-button dashed size="tiny"
                      @click="handleYamlLintCheck">
                YAML-LINT校验
            </n-button>
            <n-button dashed size="tiny">
                HELM-LINT校验
            </n-button>
            <n-button dashed size="tiny">
                KUBE-LINT校验
            </n-button>
            <n-divider vertical>

            </n-divider>
            <n-button dashed size="tiny">
                新增chart模板
            </n-button>
            <n-button dashed size="tiny">
                打包
            </n-button>
            <n-button dashed size="tiny">
                value刷新
            </n-button>

            <n-select
                    placeholder="选择主题"
                    style="display: flex;float: right;width: 150px"
                    v-model:value="themeValue" size="tiny" :options="themeOptions"/>

        </n-space>

        <n-alert v-show="checkErrorMsg" style="margin-top: 10px" title="校验失败" type="error" closable
                 v-for="(item) in checkErrorMsg">
            {{item}}
        </n-alert>
        <n-divider></n-divider>
        <n-space

                vertical>
            <n-layout
                    style="height: auto"
                    has-sider>
                <n-layout-sider
                        bordered
                        collapse-mode="width"
                        :collapsed-width="10"
                        :width="270"
                        :collapsed="collapsed"
                        show-trigger
                        @collapse="collapsed = true"
                        @expand="collapsed = false"
                >
                    <n-menu
                            :options="menuOptions"
                            :default-expanded-keys="defaultExpandedKeys"
                            @update:value="handleMenuUpdate"
                    />
                </n-layout-sider>
                <n-layout>
                    <div class="in-coder-panel">
                        <textarea ref="textarea" v-model="code"></textarea>
                    </div>

                    <n-space style="margin-top:6px;background-color: hsl(0,100%,0.1)">
                        <n-button dashed size="tiny">
                            撤销
                        </n-button>
                        <n-button dashed size="tiny">
                            重入
                        </n-button>
                        <n-divider vertical/>
                        <span>
                            长度: 11
                        </span>
                        <span>
                            行数: 11
                        </span>
                        <span>
                            选择: 11
                        </span>
                    </n-space>
                </n-layout>
            </n-layout>
        </n-space>
    </n-card>
</template>

<script setup>
    import CodeMirror from 'codemirror'
    import 'codemirror/addon/lint/lint'
    import 'codemirror/addon/lint/lint.css'
    import 'codemirror/theme/rubyblue.css'
    import 'codemirror/lib/codemirror.css'
    import 'codemirror/mode/yaml/yaml.js'
    import 'codemirror/addon/lint/yaml-lint.js'
    import jsYaml from 'js-yaml'
    import {h, onMounted, ref} from 'vue'
    import {NIcon} from "naive-ui"
    import {FolderOpenFilled, AttachFileOutlined, UnarchiveSharp} from '@vicons/material'

    function renderIcon(icon, color) {
        return () => h(NIcon, {color: color}, {default: () => h(icon)});
    }

    window.jsyaml = jsYaml

    const code = ref("欢迎使用");
    const collapsed = ref(false);
    const textarea = ref(null);
    let coder;

    const options = {
        mode: "yaml",
        // 缩进格式
        tabSize: 2,
        // 主题，对应主题库 JS 需要提前引入
        theme: "default",
        // 行号码
        lineNumbers: true,
        line: true,
        // extraKeys: {'Ctrl': 'autocomplete'},//自定义快捷键
        readOnly: false,
        lint: true,
        spellcheck: true,  // 拼写检查
        gutters: ["CodeMirror-lint-markers"],
        foldGutter: true,
        // 光标背景行高亮
        styleActiveLine: true,
        // 自动刷新
        autoRefresh: true,
    }

    onMounted(() => {
        coder = CodeMirror.fromTextArea(textarea.value, options);
        coder.on('change', coder => {
            code.value = coder.getValue();
        })

    })

    function calleArr() {
        for (let i = 0; i < menuOptions.value.length; i++) {
            var data = menuOptions.value[i];
            if (data.children) {
                calleArr(data.children)
            }
        }
    }

    const Fn = (data, key) => {
        data.forEach((item) => {
            if (item.children) {
                Fn(item.children, key)
            } else {
                if (item.key === key) {
                    item.icon = renderIcon(AttachFileOutlined, "red");
                }
            }
        })
    }

    const checkYaml = (s) => {
        jsYaml.load(s)
    }

    const checkErrorMsg = ref([]);
    const handleYamlLintCheck = () => {
        fileList.value.forEach(function (el) {
            let rd = new FileReader();
            rd.onload = (efile) => {
                try {
                    checkYaml(efile.target.result);
                } catch (e) {
                    checkErrorMsg.value.push(el.file.name + " 校验失败: " + e);
                    Fn(menuOptions.value, el.fullPath)
                }
            };
            rd.readAsBinaryString(el.file);
        })
    }

    const menuOptions = ref([]);

    const defaultExpandedKeys = ref([]);


    const fileList = ref([]);

    const handleFileChange = (file) => {
        handelChangeMenu(file.fileList);
    }

    const handleBeforeUpload = () => {
        fileList.value = [];
    }

    function changCt(arr) {
        const treeArr = [];
        arr.forEach(item => {
            const arr1 = item.split('/')
            let key = ''
            arr1.forEach((ele, index) => {
                if (ele === '') {
                    return
                }
                key = key + '/' + ele
                let obj = {}
                if (index < arr1.length - 1) {
                    obj = {
                        label: ele,
                        type: 'directory',
                        key
                    }
                } else {
                    obj = {
                        label: ele,
                        type: 'file',
                        key
                    }
                }
                treeArr.push(obj)
            })
        })
        const arr2 = treeArr.reduce((pre, cur) => {
            if (!pre.has(cur.key)) {
                pre.set(cur.key, cur)
            }
            return pre
        }, new Map())
        const arr3 = [...arr2.values()]
        return arr3
    }

    let filePathSet = new Set();

    function treeify(files) {
        files = files.reduce(function (tree, f) {
            var dir = ''
            f.key.split('/').forEach((item, index, arra) => {
                    if (item === '') {
                        return
                    }
                    if (index > 0 && index < arra.length - 1) {
                        dir = dir + '/' + item
                    }
                }
            )
            if (tree[dir]) {
                tree[dir].children.push(f)
            } else {
                tree[dir] = {implied: true, children: [f]}
            }
            if (tree[f.key]) {
                f.children = tree[f.key].children
            } else {
                if (f.type !== 'file') {
                    f.children = []
                    filePathSet.add(f.key);
                    f.icon = renderIcon(FolderOpenFilled, "green");
                } else {
                    f.icon = renderIcon(AttachFileOutlined, "green");
                }
            }

            return (tree[f.key] = f), tree
        }, {})

        return Object.keys(files).reduce(function (tree, f) {
            if (files[f].implied) {
                return tree.concat(files[f].children)
            }
            return tree
        }, [])
    }

    function compose(fn1, fn2) {
        return (...args) => fn2(fn1(...args))
    }

    const changeCtToTree = compose(changCt, treeify)

    const handelChangeMenu = (files) => {
        let arr = [];
        files.forEach(function (e) {
            arr.push(e.fullPath);
        })
        menuOptions.value = changeCtToTree(arr);
        filePathSet.forEach(function (e) {
            defaultExpandedKeys.value.push(e);
        })
    }

    const handleMenuUpdate = (v) => {
        let found = false
        fileList.value.forEach(function (el) {
            if (el.fullPath === v) {
                found = true
                let rd = new FileReader();
                rd.onload = (e) => {
                    code.value = e.target.result;
                    coder.setValue(e.target.result);
                };
                rd.readAsBinaryString(el.file);
            }
        })
        if (!found) {
            code.value = "";
            coder.setValue("");
        }
    }

</script>
<style scoped lang="scss">
    .in-coder-panel {
        width: 100%;
        height: 100%;

        :deep(.CodeMirror) {
            border: 1px solid #eee;
            height: 100%;
            width: 100%;

            .CodeMirror-code {
                line-height: 19px;
            }
        }
    }
</style>

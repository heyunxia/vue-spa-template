<template>
    <script :id="randomId" name="content" type="text/plain"></script>
</template>

<script>
    export default {
        name: 'VueUEditor',
        props: {
            ueditorPath: {
                // UEditor 代码的路径
                type: String,
                default: '/static/ue/'
            },
            ueditorConfig: {
                // UEditor 配置项
                type: Object,
                default: function () {
                    return {};
                }
            }
        },
        data () {
            return {
                // 为了避免麻烦，每个编辑器实例都用不同的 id
                randomId: 'editor_' + (Math.random() * 100000000000000000),
                instance: null,
                // scriptTagStatus -> 0:代码未加载，1:两个代码依赖加载了一个，2:两个代码依赖都已经加载完成
                scriptTagStatus: 0
            };
        },
        created () {
            if (window.UE !== undefined) {
                // 如果全局对象存在，说明编辑器代码已经初始化完成，直接加载编辑器
                this.scriptTagStatus = 2;
                this.initEditor();


            } else {
                // 如果全局对象不存在，说明编辑器代码还没有加载完成，需要加载编辑器代码
                this.insertScriptTag();
            }


        },
        beforeDestroy () {
            // 组件销毁的时候，要销毁 UEditor 实例
            if (this.instance !== null && this.instance.destroy) {
                this.instance.destroy();
            }
        },
        methods: {
            insertScriptTag () {
                let editorScriptTag = document.getElementById('editorScriptTag');
                let configScriptTag = document.getElementById('configScriptTag');
                // 如果这个tag不存在，则生成相关代码tag以加载代码
                if (editorScriptTag === null) {
                    configScriptTag = document.createElement('script');
                    configScriptTag.type = 'text/javascript'; configScriptTag.src = this.ueditorPath + 'ueditor.config.js'; configScriptTag.id = 'configScriptTag';
                    editorScriptTag = document.createElement('script');
                    editorScriptTag.type = 'text/javascript'; editorScriptTag.src = this.ueditorPath + 'ueditor.all.js'; editorScriptTag.id = 'editorScriptTag';
                    let s = document.getElementsByTagName('head')[0];
                    s.appendChild(configScriptTag);
                    s.appendChild(editorScriptTag);
                }
                // 等待代码加载完成后初始化编辑器
                if (configScriptTag.loaded) {
                    this.scriptTagStatus++;
                } else {
                    configScriptTag.addEventListener('load', () => {
                        this.scriptTagStatus++;
                        configScriptTag.loaded = true;
                        this.initEditor();
                    });
                }
                if (editorScriptTag.loaded) {
                    this.scriptTagStatus++;
                } else {
                    editorScriptTag.addEventListener('load', () => {
                        this.scriptTagStatus++;
                        editorScriptTag.loaded = true;
                        this.initEditor();
                    });
                }
                this.initEditor();
            },
            initEditor () {
                // scriptTagStatus 为 2 的时候，说明两个必需引入的 js 文件都已经被引入，且加载完成
                if (this.scriptTagStatus === 2 && this.instance === null) {
                    // Vue 异步执行 DOM 更新，这样一来代码执行到这里的时候可能 template 里面的 script 标签还没真正创建
                    // 所以，我们只能在 nextTick 里面初始化 UEditor
                    this.$nextTick(() => {
                        this.instance = window.UE.getEditor(this.randomId, this.ueditorConfig);

                        // 自定义的普通按钮
                        window.UE.registerUI('button',function(editor,uiName){

                            //注册按钮执行时的command命令，使用命令默认就会带有回退操作
                            editor.registerCommand(uiName, {
                                execCommand: function() {
                                    alert('execCommand:' + uiName)
                                }
                            });
                            //创建一个button
                            var btn = new UE.ui.Button({
                                //按钮的名字
                                name: uiName,
                                //提示
                                title: uiName,
                                //添加额外样式，指定icon图标，这里默认使用一个重复的icon
                                cssRules: 'background-position: -200px 0;',
                                //点击时执行的命令
                                onclick: function() {
                                    //这里可以不用执行命令,做你自己的操作也可
                                    editor.execCommand(uiName);
                                }
                            });
                            //当点到编辑内容上时，按钮要做的状态反射
                            editor.addListener('selectionchange', function() {
                                var state = editor.queryCommandState(uiName);
                                if (state == -1) {
                                    btn.setDisabled(true);
                                    btn.setChecked(false);
                                } else {
                                    btn.setDisabled(false);
                                    btn.setChecked(state);
                                }
                            });
                            //因为你是添加button,所以需要返回这个button
                            return btn;
                        });

                        window.UE.registerUI('skip',function(editor,uiName){

                            //创建dialog
                            var dialog = new UE.ui.Dialog({
                                //指定弹出层中页面的路径，这里只能支持页面,因为跟addCustomizeDialog.js相同目录，所以无需加路径
                                iframeUrl:'/static/ue/dialogs/skip/skip.html',
                                // iframeUrl:'/src/components/skip.vue',
                                //需要指定当前的编辑器实例
                                editor:editor,
                                //指定dialog的名字
                                name:uiName,
                                //dialog的标题
                                title:"这是个测试浮层",

                                //指定dialog的外围样式
                                cssRules:"width:600px;height:300px;",

                                //如果给出了buttons就代表dialog有确定和取消
                                buttons:[
                                    {
                                        className:'edui-okbutton',
                                        label:'确定',
                                        onclick:function () {
                                            dialog.close(true);
                                        }
                                    },
                                    {
                                        className:'edui-cancelbutton',
                                        label:'取消',
                                        onclick:function () {
                                            dialog.close(false);
                                        }
                                    }
                                ]});

                            //参考addCustomizeButton.js
                            var btn = new UE.ui.Button({
                                name:'dialogbutton' + uiName,
                                title:'dialogbutton' + uiName,
                                //需要添加的额外样式，指定icon图标，这里默认使用一个重复的icon
                                cssRules :'background-position: -500px 0;',
                                onclick:function () {
                                    //渲染dialog
                                    dialog.render();
                                    dialog.open();
                                }
                            });

                            return btn;
                        }/*index 指定添加到工具栏上的那个位置，默认时追加到最后,editorId 指定这个UI是那个编辑器实例上的，默认是页面上所有的编辑器都会添加这个按钮*/);


                        // 绑定事件，当 UEditor 初始化完成后，将编辑器实例通过自定义的 ready 事件交出去
                        this.instance.addListener('ready', () => {
                            this.$emit('ready', this.instance);
                        });
                    });
                }
            }
        }
    };
</script>
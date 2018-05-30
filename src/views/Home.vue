<template>
    <my-page title="XML 编辑器">
        <textarea id="text" class="form-control" v-model="text" rows="16" placeholder="输入 XML"></textarea>
        <div class="btns">
            <ui-raised-button class="btn" label="格式化" primary @click="format" />
            <ui-raised-button class="btn" label="压缩" @click="compress" />
            <ui-raised-button class="btn btn-copy" label="复制" />
            <ui-raised-button class="btn" label="来个测试用例" @click="showCase" />
            <ui-raised-button class="btn" label="清空结果" @click="clear" />
        </div>
    </my-page>
</template>

<script>
    /* eslint-disable */

    function formatXml(text) {
        //去掉多余的空格
        text = '\n' + text.replace(/(<\w+)(\s.*?>)/g, function ($0, name, props) {
                return name + ' ' + props.replace(/\s+(\w+=)/g, " $1");
            }).replace(/>\s*?</g, ">\n<");

        //把注释编码
        text = text.replace(/\n/g, '\r').replace(/<!--(.+?)-->/g, function ($0, text) {
            var ret = '<!--' + escape(text) + '-->';
            //alert(ret);
            return ret;
        }).replace(/\r/g, '\n');

        //调整格式
        var rgx = /\n(<(([^\?]).+?)(?:\s|\s*?>|\s*?(\/)>)(?:.*?(?:(?:(\/)>)|(?:<(\/)\2>)))?)/mg;
        var nodeStack = [];
        var output = text.replace(rgx, function ($0, all, name, isBegin, isCloseFull1, isCloseFull2, isFull1, isFull2) {
            var isClosed = (isCloseFull1 == '/') || (isCloseFull2 == '/' ) || (isFull1 == '/') || (isFull2 == '/');
            //alert([all,isClosed].join('='));
            var prefix = '';
            if (isBegin == '!') {
                prefix = getPrefix(nodeStack.length);
            }
            else {
                if (isBegin != '/') {
                    prefix = getPrefix(nodeStack.length);
                    if (!isClosed) {
                        nodeStack.push(name);
                    }
                }
                else {
                    nodeStack.pop();
                    prefix = getPrefix(nodeStack.length);
                }

            }
            var ret = '\n' + prefix + all;
            return ret;
        });

        var prefixSpace = -1;
        var outputText = output.substring(1);
        //alert(outputText);

        //把注释还原并解码，调格式
        outputText = outputText.replace(/\n/g, '\r').replace(/(\s*)<!--(.+?)-->/g, function ($0, prefix, text) {
            //alert(['[',prefix,']=',prefix.length].join(''));
            if (prefix.charAt(0) == '\r')
                prefix = prefix.substring(1);
            text = unescape(text).replace(/\r/g, '\n');
            var ret = '\n' + prefix + '<!--' + text.replace(/^\s*/mg, prefix) + '-->';
            //alert(ret);
            return ret;
        });

        return outputText.replace(/\s+$/g, '').replace(/\r/g, '\r\n');
        }
        function getPrefix(prefixIndex) {
        var span = '    ';
        var output = [];
        for (var i = 0; i < prefixIndex; ++i) {
            output.push(span);
        }

        return output.join('');
    }

    export default {
        data () {
            return {
                text: ''
            }
        },
        mounted() {
            this.clipboard = new Clipboard('.btn-copy', {
                text: trigger => {
                    return this.text
                }
            });
            this.clipboard.on('success', function(e) {
                console.info('Action:', e.action);
                console.info('Text:', e.text);
                console.info('Trigger:', e.trigger);

                e.clearSelection();
            });

            this.clipboard.on('error', function(e) {
                console.error('Action:', e.action);
                console.error('Trigger:', e.trigger);
            });
        },
        destoryed() {
            this.clipboard.destroy()
        },
        methods: {
            format() {
                $('#text').format({method: 'xml'})
                // this.text = formatXml(this.text)
            },
            compress() {
                $('#text').format({method: 'xmlmin'})
            },
            showCase() {
                this.text = `<?xml version="1.0" encoding="UTF-8"?>
<user>
    <name>Xiao Ming</name>
    <age>18</age>
</user>`
            },
            clear() {
                this.text = ''
            }
        }
    }
</script>

<style lang="scss" scoped>
    .form-control {
        margin-bottom: 16px;
    }
    .form-control:focus {
        border-color: #009688;
        outline: 0;
    }
    textarea.form-control {
        height: auto;
    }
    .form-control {
        display: block;
        width: 100%;
        height: 33px;
        padding: 6px 12px;
        font-size: 14px;
        line-height: 1.42;
        color: #55595c;
        background-color: #fff;
        background-image: none;
        border: 1px solid #ccc;
        border-radius: 2px;
    }
    .btns {
        .btn {
            margin-right: 8px;
            margin-bottom: 16px;
        }
    }
</style>

/**
* Tencent is pleased to support the open source community by making 蓝鲸智云PaaS平台社区版 (BlueKing PaaS Community
* Edition) available.
* Copyright (C) 2017-2020 THL A29 Limited, a Tencent company. All rights reserved.
* Licensed under the MIT License (the "License"); you may not use this file except in compliance with the License.
* You may obtain a copy of the License at
* http://opensource.org/licenses/MIT
* Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
* an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
* specific language governing permissions and limitations under the License.
*/
<template>
    <div class="tag-tree">
        <el-tree
            v-if="Array.isArray(value)"
            v-loading="loading"
            ref="tree"
            node-key="id"
            :data="items"
            :props="default_props"
            :show-checkbox="editable && formMode && show_checkbox"
            :default-expand-all="default_expand_all"
            :default-checked-keys="value"
            :default-expanded-keys="expanded_keys"
            :filter-node-method="filterNode"
            :check-on-click-node="true"
            :expand-on-click-node="false"
            @check="_handleBoxCheck"
            @node-expand="nodeExpend"
            @node-collapse="nodeCollapse">
        </el-tree>
        <span v-show="!validateInfo.valid" class="common-error-tip error-info">{{validateInfo.message}}</span>
    </div>
</template>
<script>
    import '@/utils/i18n.js'
    import { getFormMixins } from '../formMixins.js'

    export const attrs = {
        value: {
            type: [Array, String],
            required: false,
            default () {
                return []
            }
        },
        items: {
            type: Array,
            required: false,
            default () {
                return []
            },
            desc: 'set tree data, which should be a multiple layers array'
        },
        expanded_keys: {
            type: Array,
            required: false,
            default () {
                return []
            }
        },
        show_checkbox: {
            type: Boolean,
            required: false,
            default: true,
            desc: 'whether node is selectable'
        },
        default_expand_all: {
            type: Boolean,
            required: false,
            default: true,
            desc: 'whether to expand all nodes by default'
        },
        remote: {
            type: Boolean,
            required: false,
            default: false,
            desc: 'use remote data or not'
        },
        remote_url: {
            type: [String, Function],
            required: false,
            default: '',
            desc: 'remote url when remote is true'
        },
        remote_data_init: {
            type: Function,
            required: false,
            default: function (data) {
                return data
            },
            desc: 'how to process data after getting remote data'
        }
    }
    export default {
        name: 'TagTree',
        mixins: [getFormMixins(attrs)],
        data () {
            return {
                default_props: {
                    label: 'label',
                    children: 'children',
                    disabled: true
                },
                loading: false
            }
        },
        mounted () {
            this.remoteMethod()
        },
        methods: {
            handleCheckChange (data, checked, indeterminate) {
                return true
            },
            filterNode (value, data) {
                return true
            },
            handleBoxCheck (data) {
                return this.editable && this.formMode && this.show_checkbox
            },
            _handleBoxCheck (data) {
                if (this.handleBoxCheck(data)) {
                    this.updateForm(this.$refs.tree.getCheckedKeys(true))
                }
            },
            nodeExpend (data, node, tag) {
                this.expanded_keys.push(data.id)
            },
            nodeCollapse (data, node, tag) {
                this.expanded_keys.pop(data.id)
            },
            remoteMethod () {
                const self = this

                const remote_url = typeof this.remote_url === 'function' ? this.remote_url() : this.remote_url
                if (!remote_url) return

                this.loading = true

                // 请求远程数据
                $.ajax({
                    url: remote_url,
                    method: 'GET',
                    success: function (res) {
                        let treeData = self.remote_data_init(res)
                        // 表单为展示模式时，去掉未被选中的数据项
                        if (!self.editable || !self.formMode) {
                            treeData = self.filterTreeItem(treeData)
                        }

                        self.items = treeData
                        self.$refs.tree && self.$refs.tree.setCheckedKeys(self.value) // 兼容组件勾选的情况

                        self.loading = false
                    },
                    error: function () {
                        self.empty_text = gettext('请求数据失败')
                        self.loading = false
                    }
                })
            },
            filterTreeItem (data) {
                return data.filter(item => {
                    if (this.value.indexOf(item.id) > -1) {
                        return item
                    } else if (item.children) {
                        item.children = this.filterTreeItem(item.children)
                        if (item.children.length > 0) {
                            return item
                        }
                    }
                })
            }
        }
    }
</script>
<style lang="scss" scoped>
.tag-tree {
    /deep/ .el-tree {
        & > .el-tree-node > .el-tree-node__content {
            height: 36px;
        }
        .el-tree-node__label {
            padding-left: 4px;
        }
        .el-tree__empty-text {
            font-size: 12px;
        }
    }
}
</style>

<template>
    <el-row>
        <el-col :span="12">
            <el-form id="siteForm"
                     v-loading="loading"
                     element-loading-text="保存中..."
                     :rules="rules" ref="form" :model="form" label-width="auto" :status-icon="true">
                <el-form-item label="站点名称" prop="siteName">
                    <el-input v-model="form.siteName"/>
                </el-form-item>

                <el-form-item label="站点域名" prop="domain">
                    <el-input v-model="form.domain"/>
                </el-form-item>

                <el-form-item label="开启搜索">
                    <el-switch v-model="form.searchEnable"/>
                    <span class="zfile-word-aux">从缓存中搜索文件, 仅当开启缓存后有效！</span>
                </el-form-item>

                <el-form-item label="搜索包含加密文件">
                    <el-switch v-model="form.searchContainEncryptedFile"/>
                </el-form-item>

                <el-form-item label="搜索忽略大小写">
                    <el-switch v-model="form.searchIgnoreCase"/>
                </el-form-item>

                <el-form-item label="存储策略">
                    <el-select filterable v-model="form.storageStrategy" placeholder="请选择存储策略">
                        <el-option :key="item.key"
                                   v-for="(item) in supportStrategy"
                                   :label="item.description"
                                   :disabled="!item.available"
                                   :value="item.key">
                                <span style="float: left">{{ item.description }}</span>
                                <span style="float: right;">
                                    <el-badge v-if="item.available" value="有效" class="item" type="success"/>
                                    <el-badge v-else value="无效" class="item"/>
                                </span>
                        </el-option>
                    </el-select>
                    <el-button type="text" style="margin-left: 20px;" @click="jumpStorageStrategyConfig">设置策略</el-button>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="submitForm('form')" round>保存设置</el-button>
                </el-form-item>
            </el-form>
        </el-col>
    </el-row>
</template>

<script>
    import qs from 'qs';

    export default {
        name: "SiteSetting",
        data() {
            return {
                form: {
                    siteName: '',
                    storageStrategy: '',
                    domain: '',
                    infoEnable: false,
                    searchEnable: false,
                    searchIgnoreCase: false,
                    enableCache: false,
                    searchContainEncryptedFile: true
                },
                loading: false,
                supportStrategy: [],
                rules: {
                    siteName: [
                        {required: true, message: '请输入站点名称', trigger: 'change'},
                    ],
                    domain: [
                        {required: true, type: 'url', message: '请输入正确的域名, 需以 http:// 或 https:// 开头', trigger: 'change'},
                    ]
                }
            };
        },
        methods: {
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        this.loading = true;
                        this.$http.post('admin/config', qs.stringify(this.form)).then((response) => {
                            this.loading = false;
                            if (response.data.code === 0) {
                                this.$message({
                                    message: '保存成功',
                                    type: 'success'
                                });
                            } else {
                                this.$message({
                                    message: response.data.msg,
                                    type: 'error'
                                });
                            }
                        })
                    } else {
                        return false;
                    }
                });
            },
            jumpStorageStrategyConfig() {
                this.$router.push({path: '/admin/storage?type=' + this.form.storageStrategy});
            }
        },
        mounted() {
            this.$http.get('admin/support-strategy').then((response) => {
                this.supportStrategy = response.data.data;
            });

            this.$http.get('admin/config').then((response) => {
                this.form = response.data.data;
            });
        }
    }
</script>

<style scoped>
    .el-row {
        overflow-y: auto;
    }

    #siteForm {
        margin-top: 20px;
        margin-left: 20px;
    }

    #siteForm >>> .el-select {
        width: 70%;
    }

    .zfile-word-aux {
        margin-left: 20px;
        color: #aaaaaa;
    }
</style>
<template>
  <div>
    <Row>
      <Col span="6">
        <Card>
          <Row type="flex" justify="center" align="top" class-name="module-title-wrapper">
            <Col span="16">
              <span class="module-title">员工列表</span>
            </Col>
            <Col span="8" class="ivu-text-right">
              <Button>高级权限</Button>
            </Col>
          </Row>
          <Row :gutter="24" type="flex" justify="end" class="ivu-mt">
            <Col span="24">
              <Input
                prefix="ios-search"
                placeholder="姓名/电话/职位"
                v-model="searchKey"
                @on-change="getUserinfoList"
                clearable
              />
            </Col>
          </Row>
          <Row class="mt6 noplr">
            <Col span="23" offset="1">
              <Tree :data="treeOfUserData" @on-select-change="getChild"></Tree>
            </Col>
          </Row>
          <Row :gutter="24" type="flex" justify="end" class="mtb15">
            <Col span="24" class="ivu-text-center">
              <Button type="primary" @click="handleUserinfoAdd">+新增员工</Button>
            </Col>
          </Row>
        </Card>
      </Col>
      <Col span="18" class="box">
        <Card>
          <Row type="flex" justify="center" align="top" class-name="module-title-wrapper">
            <Col span="24">
              <span class="module-title">员工明细</span>
            </Col>
          </Row>
          <Row :gutter="24" type="flex" justify="end" class="mtb15">
            <Col span="6" class="text-center">
              <div class="user-info-detail">
                <i-link target="_blank" slot="avatar">
                  <Avatar :src="userInfo.portrait" />
                </i-link>
              </div>
              <Upload
                ref="portrait"
                v-model="userInfo.portrait"
                :action="resource"
                :headers="headers"
                :format="[ 'png', 'jpg', 'jpeg']"
                :max-size="1024*10"
                :on-success="handleHeaderUrlSuccess"
                :on-format-error="handleFileFormatErr"
                :on-exceeded-size="handleFileSizeErr"
                :before-upload="handleBeforeLicenseUrlUpload"
                :show-upload-list="false"
              >
                <Button>更换图片</Button>
              </Upload>
            </Col>
            <Col span="12">
              <Form ref="editForm" :model="userInfo" :rules="rules" :label-width="120">
                <FormItem label="所属医院">
                  <Select v-model="userInfo.hospitalId" placeholder="请选择">
                    <Option
                      v-for="(it, index) in hospitalList"
                      :key="index"
                      :value="it.id"
                    >{{ it.name }}</Option>
                  </Select>
                </FormItem>
                <FormItem label="姓名" prop="name">
                  <Input v-model="userInfo.name" placeholder="必填" />
                </FormItem>
                <FormItem label="职位" prop="position" label-for="position">
                  <Select v-model="userInfo.positionCode" placeholder="请选择" element-id="position">
                    <Option
                      v-for="(it, index) in positionList"
                      :key="index"
                      :value="it.code"
                    >{{ it.name }}</Option>
                  </Select>
                </FormItem>
                <FormItem label="电话" prop="mobile">
                  <Input v-model="userInfo.mobile" placeholder="请输入" />
                </FormItem>
                <FormItem label="密码" prop="password">
                  <Input v-model="userInfo.password" placeholder="请输入" type="password" password />
                </FormItem>
                <FormItem label="执业兽医师号" prop="code">
                  <Input v-model="userInfo.code" placeholder="请输入" />
                </FormItem>
                <FormItem label="授业恩师" prop="master">
                  <Select
                    v-model="userInfo.masterId"
                    :label-in-value="true"
                    placeholder="请选择"
                    @on-change="setMasterName"
                  >
                    <Option
                      v-for="(it, index) in masterList"
                      :key="index"
                      :value="it.id"
                    >{{ it.name }}</Option>
                  </Select>
                </FormItem>
                <FormItem label="备注">
                  <Input v-model="userInfo.remark" type="textarea" placeholder="请输入" />
                </FormItem>
              </Form>
            </Col>
            <Col span="6"></Col>
          </Row>
          <Row :gutter="24" type="flex" justify="end" class="mtb15">
            <Col span="24">
              <h4 class="mb10">权限设置列表</h4>
              <Table border :columns="columns1" :data="data1"></Table>
            </Col>
          </Row>
          <Row :gutter="24" type="flex" justify="end" class="mtb15">
            <Col span="24" class="ivu-text-right">
              <Button
                type="primary"
                @click="handleUserinfoSave"
                :disabled="currentHospitalId===''&&!this.userInfo.id?true:false"
              >{{isAdd?'添加':'保存'}}</Button>
            </Col>
          </Row>
        </Card>
      </Col>
    </Row>
  </div>
</template>
<script>
    export default {
        name: 'list-table-list',
        inject: ['reload'], // 注入App里的reload方法
        data () {
            return {
                hospitalList: [],
                masterList: [],
                treeOfUserData: [],
                resource: this.$store.state.admin.user.resource,
                headers: this.$store.state.admin.user.headers,
                searchKey: '',
                rules: {
                    name: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
                    mobile: [
                        { required: true, message: '请输入电话', trigger: 'blur' },
                        {
                            type: 'string',
                            pattern: /^\d+$/,
                            message: '请输入数字',
                            trigger: 'change'
                        }
                    ],
                    code: [
                        { required: true, message: '请输入执业兽医师号', trigger: 'blur' }
                    ],
                    remark: [
                        { max: 200, message: '备注不得超过200个字符', trigger: 'change' }
                    ]
                },
                positionList: [],
                currentId: '',
                userInfo: {
                },
                isAdd: true,
                columns1: [
                    {
                        title: '模块',
                        minWidth: 84,
                        key: 'module'
                    },
                    {
                        title: '项目名称',
                        minWidth: 84,
                        key: 'project'
                    },
                    {
                        title: '查看权限',
                        minWidth: 84,
                        key: 'view',
                        render: (h, params) => {
                            return h('div', [
                                h('checkbox', {
                                    props: {
                                        value: params.row.view
                                    },
                                    // style: {
                                    //     display: params.row.view ? 'none' : ''
                                    // },
                                    on: {
                                        'on-change': () => {
                                            // this.reviewFeedback(this.feedbackPageList[params.index].id)
                                        }
                                    }
                                })
                            ]);
                        }
                    },
                    {
                        title: '操作权限',
                        minWidth: 84,
                        key: 'edit',
                        render: (h, params) => {
                            return h('div', [
                                h('checkbox', {
                                    props: {
                                        value: params.row.edit
                                    },
                                    // style: {
                                    //     display: params.row.view ? 'none' : ''
                                    // },
                                    on: {
                                        'on-change': () => {
                                            // this.reviewFeedback(this.feedbackPageList[params.index].id)
                                        }
                                    }
                                })
                            ]);
                        }
                    }
                ],
                currentHospitalId: '',
                data1: [
                    {
                        module: '顾客中心',
                        project: '首页管理',
                        view: true,
                        edit: true
                    },
                    {
                        module: '顾客中心',
                        project: '登记办卡',
                        view: true,
                        edit: false
                    },
                    {
                        module: '顾客中心',
                        project: '首页管理',
                        view: false,
                        edit: true
                    },
                    {
                        module: '顾客中心',
                        project: '登记办卡',
                        view: false,
                        edit: false
                    },
                    {
                        module: '顾客中心',
                        project: '首页管理',
                        view: true,
                        edit: false
                    }
                ]
            };
        },
        methods: {
            getChild (data, selectedNode) {
                this.$set(selectedNode, 'expand', !selectedNode.expand); // 点击节点文字展开收起
                if (data && data.length > 0) {
                    if (selectedNode.treeType === 'hospital') {
                        this.currentHospitalId = selectedNode.id;
                        this.userInfo = {
                            hospitalId: selectedNode.id
                        };
                        this.getMasterList(selectedNode.id);
                        this.isAdd = true;
                    }
                    if (selectedNode.treeType === 'user') {
                        this.currentHospitalId = '';
                        this.userInfo = selectedNode;
                        this.getMasterList(selectedNode.hospitalId);
                        this.isAdd = false;
                    }
                } else {
                    // 取消选中
                    this.currentHospitalId = '';
                    this.userInfo = {
                    };
                }
            },
            setMasterName (obj) {
                this.userInfo.masterName = obj.label;
            },
            handleHeaderUrlSuccess (response, file, fileList) {
                this.$set(this.userInfo, 'portrait', response.data);
            },
            handleFileFormatErr (file) {
                this.$Notice.warning({
                    title: '文件类型错误',
                    desc:
                        '文件  ' + file.name + ' 文件类型错误,请上传png, jpg, jpeg等格式图片.'
                });
            },
            handleFileSizeErr (file) {
                this.$Notice.warning({
                    title: '文件过大',
                    desc: '文件  ' + file.name + ' 过大,不得超过10M.'
                });
            },
            handleBeforeLicenseUrlUpload () {
                this.$refs.portrait.clearFiles();
            },
            getUserinfoList () {
                var data = {
                    searchKey: this.searchKey
                };
                this.$get('/admin/user/hospital/page', data, response => {
                    this.treeOfUserData = [];
                    var rtn = response.data.data;
                    this.hospitalList = response.data.data;
                    // 处理左侧树数据
                    for (var i = 0; i < rtn.length; i++) {
                        var obj = {};
                        obj.title = rtn[i].name + '(' + rtn[i].userBo.data.length + ')';
                        obj.id = rtn[i].id;
                        obj.treeType = 'hospital';
                        obj.expand = false; // 树默认不展开
                        obj.minWidth = 84;
                        var childrenList = [];
                        for (var j = 0; j < rtn[i].userBo.data.length; j++) {
                            if (rtn[i].userBo.data[j].position) {
                                rtn[i].userBo.data[j].positionCode = rtn[i].userBo.data[j].position.code
                            } else {
                                rtn[i].userBo.data[j].positionCode = ''
                            }
                            if (rtn[i].userBo.data[j].master) {
                                rtn[i].userBo.data[j].masterId = rtn[i].userBo.data[j].master.id
                            } else {
                                rtn[i].userBo.data[j].masterId = ''
                            }
                            var child = {};
                            child = rtn[i].userBo.data[j];
                            child.treeType = 'user';
                            child.title =
                                rtn[i].userBo.data[j].name +
                                '(' +
                                (rtn[i].userBo.data[j].position
                                    ? rtn[i].userBo.data[j].position.name
                                : '') +
                                ')';
                            if (rtn[i].userBo.data[j].master) {
                                child.masterId = rtn[i].userBo.data[j].master.id;
                            }
                            if (
                                localStorage.currentUserIdForUserInfo &&
                                localStorage.currentUserIdForUserInfo === rtn[i].userBo.data[j].id
                            ) {
                                this.isAdd = false;
                                child.selected = true;
                                obj.expand = true;

                                this.userInfo = child;
                                this.getMasterList(child.hospitalId);
                            }
                            childrenList.push(child);
                        }
                        obj.children = childrenList;
                        this.treeOfUserData.push(obj);
                    }
                });
            },
            handleUserinfoSave () {
                if (this.isAdd && this.currentHospitalId === '') {
                    this.$Message.error('请选择左侧医院');
                    return false;
                }
                if (!this.isAdd && !this.userInfo.id) {
                    this.$Message.error('请选择左侧员工');
                    return false;
                }
                this.$refs.editForm.validate(valid => {
                    if (valid) {
                        this.userInfo.type = 'employee';
                        delete this.userInfo.position
                        delete this.userInfo.master
                        this.$post('/admin/user/save', this.userInfo, response => {
                            if (response.success) {
                                this.$Message.info('保存成功');
                                localStorage.currentUserIdForUserInfo = this.userInfo.id;
                                this.isAdd = false;
                                this.reload(); // 调用局部刷新方法
                            } else {
                                this.$Message.error(response.message);
                            }
                        });
                    } else {
                    }
                });
            },
            handleUserinfoAdd () {
                if (this.currentHospitalId === '') {
                    this.$Message.error('请选择左侧医院');
                    return false;
                }
                this.userInfo = {
                    hospitalId: this.currentHospitalId
                };
                this.isAdd = true;
            },
            getPositonList () {
                this.$get('/admin/user/position/page', {}, response => {
                    this.positionList = response.data.data;
                });
            },
            getMasterList (hospitalId) {
                this.$get('/admin/user/page?hospitalId=' + hospitalId, {}, response => {
                    this.masterList = response.data.data;
                });
            }
        },
        mounted () {
            this.getUserinfoList();
            this.getPositonList();
        }
    };
</script>
<style lang="less" scoped>
.box {
  padding-left: 15px;
}

.mtb15 {
  margin: 15px 0;
}
</style>
<style lang="less">
.user-info-list {
  height: 570px;
  overflow: auto;
}
.user-info-avatar {
  margin: 0 32px 0 16px;
}
.user-info-avatar .ivu-avatar-default {
  width: 60px;
  height: 60px;
}
.text-center {
  text-align: center;
}
.user-info-detail {
  margin: 36px 16px;
}
.user-info-detail .ivu-avatar-default {
  width: 120px;
  height: 120px;
}
</style>

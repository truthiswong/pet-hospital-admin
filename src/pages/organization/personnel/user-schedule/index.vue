<template>
  <div>
    <Row>
      <Col span="5">
        <Card>
          <Row type="flex" justify="center" align="top" class-name="module-title-wrapper">
            <Col span="24">
              <span class="module-title">班次列表</span>
            </Col>
          </Row>
          <Row class="noplr">
            <Col span="24">
              <List class="shift-list">
                <ListItem
                  v-for="(item, index) in scheduleList"
                  :key="index"
                  :class="item.id===scheduleData.id?'active':''"
                >
                  <p
                    @click="switchList(item)"
                    class="list"
                  >{{ item.name }}({{ item.beginTime }}~{{ item.endTime }})</p>
                </ListItem>
              </List>
            </Col>
          </Row>
          <Row :gutter="16" type="flex" justify="end" class="mtb15">
            <Col span="10" class="ivu-text-center">
              <Button type="success" @click="handleOpenCreate">+添加班次</Button>
            </Col>
            <Col span="7" class="ivu-text-center">
              <Button type="primary" @click="handleOpenEdit">编辑</Button>
            </Col>
            <Col span="7" class="ivu-text-center">
              <Button type="error">删除</Button>
            </Col>
          </Row>
        </Card>
      </Col>
      <Col span="19" class="box">
        <Card>
          <Row type="flex" justify="center" align="top" class-name="module-title-wrapper">
            <Col span="24">
              <span class="module-title">排班表</span>
            </Col>
          </Row>
          <Row :gutter="16" type="flex" justify="end" class="mtb15">
            <Col span="24">
              <table class="table">
                <thead>
                  <tr>
                    <!-- <th>职位</th> -->
                    <th>人员</th>
                    <th>周一</th>
                    <th>周二</th>
                    <th>周三</th>
                    <th>周四</th>
                    <th>周五</th>
                    <th>周六</th>
                    <th>周日</th>
                  </tr>
                </thead>
                <!-- 没有职位的tbody--暂时 -->
                <tbody v-if="memberList.length>0">
                  <tr v-for="(item, index) in memberList" :key="index">
                    <td>{{item.userName}}</td>
                    <template v-for="(day) in weekDay">
                      <td @click="changeArrange(index,day)" :key="day">
                        <Icon v-if="item[day]" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                    </template>
                  </tr>
                </tbody>
                <tbody v-else>
                  <tr>
                    <td colspan="8">暂无数据</td>
                  </tr>
                </tbody>
                <!-- 有职位的tbody-等返回的数据有职位时再放出来 -->
                <!-- <tbody v-for="(item, index) in memberList" :key="index">
                  <tr>
                    <td :rowspan="item.list.length">{{item.position}}</td>
                    <td>{{item.list[0].name}}</td>
                    <td @click="item.list[0].monday=!item.list[0].monday">
                      <Icon v-if="item.list[0].monday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                    <td @click="item.list[0].tuesday=!item.list[0].tuesday">
                      <Icon v-if="item.list[0].tuesday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                    <td @click="item.list[0].wednesday=!item.list[0].wednesday">
                      <Icon v-if="item.list[0].wednesday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                    <td @click="item.list[0].thursday=!item.list[0].thursday">
                      <Icon v-if="item.list[0].thursday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                    <td @click="item.list[0].friday=!item.list[0].friday">
                      <Icon v-if="item.list[0].friday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                    <td @click="item.list[0].saturday=!item.list[0].saturday">
                      <Icon v-if="item.list[0].saturday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                    <td @click="item.list[0].sunday=!item.list[0].sunday">
                      <Icon v-if="item.list[0].sunday" type="md-checkmark-circle" size="25" />
                      <Icon v-else type="md-close-circle" size="25" />
                    </td>
                  </tr>
                  <template v-if="item.list.length>1" v-for="(item2,index2) in item.list">
                    <tr v-if="index2>0" :key="index2">
                      <td>{{item2.name}}</td>
                      <td @click="item2.monday=!item2.monday">
                        <Icon v-if="item2.monday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                      <td @click="item2.tuesday=!item2.tuesday">
                        <Icon v-if="item2.tuesday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                      <td @click="item2.wednesday=!item2.wednesday">
                        <Icon v-if="item2.wednesday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                      <td @click="item2.thursday=!item2.thursday">
                        <Icon v-if="item2.thursday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                      <td @click="item2.friday=!item2.friday">
                        <Icon v-if="item2.friday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                      <td @click="item2.saturday=!item2.saturday">
                        <Icon v-if="item2.saturday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                      <td @click="item2.sunday=!item2.sunday">
                        <Icon v-if="item2.sunday" type="md-checkmark-circle" size="25" />
                        <Icon v-else type="md-close-circle" size="25" />
                      </td>
                    </tr>
                  </template>
                </tbody>-->
              </table>
              <div class="ivu-mt ivu-text-right">
                <Page :total="memberList.length" :current.sync="current" />
              </div>
            </Col>
          </Row>
          <Row :gutter="16" type="flex" justify="end" class="mtb15">
            <Col span="24" class="ivu-text-left">
              <Button type="warning" @click="saveUserSchedule">保存</Button>
            </Col>
          </Row>
        </Card>
      </Col>
    </Row>
    <Modal ref="creatModal" v-model="showCreate" @on-ok="handleCreate" :loading="true">
      <div slot="header">{{isAddSchedule?'添加班次':'编辑班次'}}</div>
      <Form ref="createForm" :model="scheduleData" :rules="scheduleRules" :label-width="170">
        <FormItem label="班次名称：" prop="name">
          <Input style="width:194px" v-model="scheduleData.name" />
        </FormItem>
        <FormItem label="上班时间：" prop="beginTime">
          <TimePicker
            :steps="[1, 5]"
            format="HH:mm"
            v-model="scheduleData.beginTime"
            :disabled-hours="startHoursDis"
            @on-change="startTimeChange"
          ></TimePicker>
        </FormItem>
        <FormItem label="下班时间：" prop="endTime">
          <TimePicker
            :steps="[1, 5]"
            format="HH:mm"
            v-model="scheduleData.endTime"
            :disabled-hours="endHoursDis"
            @on-change="endTimeChange"
          ></TimePicker>
        </FormItem>
      </Form>
    </Modal>
  </div>
</template>
<script>
    export default {
        data () {
            return {
                weekDay: [
                    'monday',
                    'tuesday',
                    'wednesday',
                    'thursday',
                    'friday',
                    'saturday',
                    'sunday'
                ],
                startHoursDis: [],
                endHoursDis: [],
                allHours: [],
                isAddSchedule: false,
                scheduleData: {
                    id: '',
                    name: '',
                    beginTime: '',
                    endTime: ''
                },
                scheduleRules: {
                    name: [
                        { required: true, message: '请输入班次名称', trigger: 'change' }
                    ],
                    beginTime: [
                        { required: true, message: '请输入上班时间', trigger: 'change' }
                    ],
                    endTime: [
                        { required: true, message: '请输入下班时间', trigger: 'change' }
                    ]
                },
                scheduleList: [],
                memberList: [
                    // {
                    //     position: '护士',
                    //     list: [
                    //         {
                    //             name: '张三',
                    //             monday: true,
                    //             tuesday: true,
                    //             wednesday: true,
                    //             thursday: true,
                    //             friday: true,
                    //             saturday: true,
                    //             sunday: true
                    //         },
                    //         {
                    //             name: '李四',
                    //             monday: true,
                    //             tuesday: true,
                    //             wednesday: true,
                    //             thursday: true,
                    //             friday: true,
                    //             saturday: true,
                    //             sunday: true
                    //         }
                    //     ]
                    // }
                ],
                current: 1,
                showCreate: false
            };
        },
        methods: {
            setAllHours () {
                for (var i = 0; i < 24; i++) {
                    this.allHours.push(i);
                }
            },
            startTimeChange (e) {
                let ar = this.allHours.filter(function (elem) {
                    return elem < Number(e.split(':')[0]);
                });
                this.endHoursDis = ar;
            },
            endTimeChange (e) {
                let ar = this.allHours.filter(function (elem) {
                    return elem > Number(e.split(':')[0]);
                });
                this.startHoursDis = ar;
            },
            handleOpenCreate () {
                this.isAddSchedule = true;
                this.$refs.createForm.resetFields();
                this.scheduleData = {};
                this.startHoursDis = [];
                this.endHoursDis = [];
                this.showCreate = true;
            },
            handleOpenEdit () {
                this.isAddSchedule = false;
                let vm = this;
                let ar = this.allHours.filter(function (elem) {
                    return elem < Number(vm.scheduleData.beginTime.split(':')[0]);
                });
                this.endHoursDis = ar;
                let er = this.allHours.filter(function (elem) {
                    return elem > Number(vm.scheduleData.endTime.split(':')[0]);
                });
                this.startHoursDis = er;
                this.showCreate = true;
            },
            handleCreate () {
                this.$refs.creatModal.buttonLoading = false;
                this.$refs.createForm.validate(valid => {
                    if (valid) {
                        this.$post(
                            '/admin/general/schedule/save',
                            this.scheduleData,
                            response => {
                                if (response.success) {
                                    this.$Message.info('保存成功');
                                    this.getScheduleList();
                                    this.showCreate = false;
                                }
                            }
                        );
                    } else {
                    }
                });
            },
            saveUserSchedule () {
                // 处理数据后保存
                for (var i = 0; i < this.memberList.length; i++) {
                    var obj = {}
                    obj.id = this.memberList[i].id
                    obj.userId = this.memberList[i].userId
                    obj.scheduleId = this.memberList[i].scheduleId
                    var weekDayStr = ''
                    var weekDayArrange = ''
                    for (var z = 0; z < this.weekDay.length; z++) {
                        if (
                            this.memberList[i][this.weekDay[z]]
                        ) {
                            weekDayStr += weekDayStr === '' ? this.weekDay[z] : ',' + this.weekDay[z]
                            weekDayArrange += weekDayArrange === '' ? 1 : ',' + 1
                        }
                    }
                    obj.weekDay = weekDayStr
                    obj.arrange = weekDayArrange
                    this.$post(
                        '/admin/user/schedule/save',
                        obj,
                        response => {
                            if (response.success) {
                            }
                        }
                    )
                }
                this.$Message.info('保存成功');
            },
            switchList (item) {
                this.scheduleData = JSON.parse(JSON.stringify(item));
                this.getUserScheduleList()
            },
            getScheduleList () {
                this.$get('/admin/general/schedule/page', {}, response => {
                    this.scheduleList = response.data.data;
                    this.scheduleData =
                        response.data.data &&
                        JSON.parse(JSON.stringify(response.data.data[0]));
                });
            },
            getUserScheduleList () {
                var data = {
                    scheduleId: this.scheduleData.id
                }
                this.$get('/admin/user/schedule/page', data, response => {
                    this.memberList = response.data.data;
                    if (response.data.data.length === 0) {
                        return false
                    }
                    // 处理表格数据
                    for (var i = 0; i < this.memberList.length; i++) {
                        for (var j = 0; j < this.memberList[i].details.length; j++) {
                            for (var z = 0; z < this.weekDay.length; z++) {
                                if (
                                    this.memberList[i].details[j].weekDay.code === this.weekDay[z]
                                ) {
                                    this.memberList[i][this.weekDay[z]] = true;
                                }
                            }
                        }
                    }
                });
            },
            changeArrange (index, day) {
                this.memberList[index][day] = !this.memberList[index][day]
                console.log(this.memberList)
                this.$forceUpdate()
            }
        },
        mounted () {
            this.getScheduleList();
            this.setAllHours();
            setTimeout(() => {
                // 设置延迟执行
                this.getUserScheduleList();
            }, 800)
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
.table {
  width: 100%;
  max-width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
  thead tr {
    background-color: #f8f8f9;
    background-image: -webkit-gradient(
      linear,
      0 0,
      0 100%,
      from(#f2f2f2),
      to(#fafafa)
    );
    background-image: -webkit-linear-gradient(top, #f2f2f2 0, #fafafa 100%);
    background-image: -moz-linear-gradient(top, #f2f2f2 0, #fafafa 100%);
    background-image: -ms-linear-gradient(top, #f2f2f2 0, #fafafa 100%);
    background-image: -o-linear-gradient(top, #f2f2f2 0, #fafafa 100%);
    background-image: -linear-gradient(top, #f2f2f2 0, #fafafa 100%);
    font-size: 12px;
  }
  th,
  td {
    padding: 8px 6px;
    vertical-align: middle;
    border: 1px solid #ddd;
    word-break: break-word;
    word-wrap: break-word;
    line-height: 1.42857143;
    font-size: 12px;
    text-align: center;
  }
}
</style>
<style lang="less">
.shift-list {
  height: 265px;
  overflow: auto;
}
</style>

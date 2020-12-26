<template>
  <div>
    <el-container direction="horizontal">
      <el-main>
        <el-tabs :tab-position="top" style="height: 100%;">
          <el-tab-pane label="未完成">
            <el-table :data="currentTasks" style="width: 100%"  stripe="true" :cell-style="tableColor">
              <el-table-column label="任务内容" width="140" align="center">
                <template slot-scope="scope">
                  <span>{{scope.row.title}}</span>
                </template></el-table-column>
              <el-table-column  label="起始时间" width="150" align="center" :filters="[{text: '今天', value: getCurrentDate()}]"
                               :filter-method="filterHandler">
                <template slot-scope="scope">
                  <span>{{scope.row.startTime | formatDate}}</span>
                </template>
              </el-table-column>
              <el-table-column  label="截止时间" width="150" align="center" :filters="[{text: '今天', value: getCurrentDate()}]"
                               :filter-method="filterHandler">
                <template slot-scope="scope">
                  <span>{{scope.row.endTime | formatDate}} </span>
                </template>
              </el-table-column>
              <el-table-column prop="importance" label="重要程度" width="120" align="center"></el-table-column>
              <el-table-column prop="completion" sortable label="完成度" width="180" align="center">
                <template slot-scope="scope">
                  <div class="block" v-if="scope.row.completion!==100">
                    <el-slider v-model="scope.row.completion" :format-tooltip="formatTooltip" class="slider-diy"></el-slider>
                  </div>
                  <div class="completion_div" v-else-if="scope.row.completion===100">
                    <img src="./assets/check-circle.jpg" alt="uuu" />
                  </div>
                </template>
              </el-table-column>
              <el-table-column prop="completion" label="完成情况" width="150" align="center">
                <template slot-scope="scope">
                  <span>{{ getTaskDone(scope.row.completion) }}</span>
                </template>
              </el-table-column>
              <el-table-column prop="operator" width="270" align="center">
                <template slot-scope="scope">
                  <el-button-group>
                    <el-button type="primary" @click="edit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="success" @click="finishTask(scope.$index, scope.row)">完成</el-button>
                    <el-button type="danger" @click="deleteTask(scope.$index, scope.row)">删除</el-button>
                  </el-button-group>
                </template>
              </el-table-column>
            </el-table>
            <el-row>
              <el-button type="primary" style="width:100%" @click="addTasks()">添加任务</el-button>
              <el-dialog title="添加任务" :visible.sync="addDialogVisible">
                <el-form ref="addTaskForm" :model="addTask" label-width="100px">
                  <el-form-item label="任务标题">
                    <el-input placeholder="请输入任务标题" v-model="addTask.title"></el-input>
                  </el-form-item>
                  <el-form-item label="任务开始时间">
                    <el-col :span="11">
                      <el-date-picker type="date" placeholder="选择日期" v-model="addTask.startTime" style="width: 100%;"></el-date-picker>
                    </el-col>
                  </el-form-item>
                  <el-form-item label="任务截至时间">
                    <el-col :span="11">
                      <el-date-picker type="date" placeholder="选择日期" v-model="addTask.endTime" style="width: 100%;" format="yyyy-MM-dd"></el-date-picker>
                    </el-col>
                  </el-form-item>
                  <el-form-item label="任务重要程度">
                    <el-select v-model="addTask.importance" placeholder="请选择任务重要程度">
                      <el-option label="普通任务" value="普通任务"></el-option>
                      <el-option label="重要任务" value="重要任务"></el-option>
                    </el-select>
                  </el-form-item>
                  <el-form-item label="任务完成度">
                    <el-slider v-model="addTask.completion"></el-slider>
                  </el-form-item>
                  <el-form-item label="任务完成情况">
                    <el-input placeholder="请输入任务情况描述" v-model="addTask.state"></el-input>
                  </el-form-item>
                  <el-form-item label="任务内容">
                    <el-input type="textarea" v-model="addTask.content"></el-input>
                  </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                  <el-button @click="addDialogVisible = false">取 消</el-button>
                  <el-button type="primary" @click="confirm('addTaskForm')">确 定</el-button>
                </div>
              </el-dialog>
            </el-row>
          </el-tab-pane>
          <el-dialog title="编辑任务" :visible.sync="editDialogVisible">
            <el-form ref="editTasks" :model="editTask" label-width="100px">
              <el-form-item label="任务标题">
                <el-input placeholder="请输入任务内容" v-model="editTask.title"></el-input>
              </el-form-item>
              <el-form-item label="起始时间">
                <el-col :span="11">
                  <el-date-picker type="date" placeholder="选择日期" v-model="editTask.startTime" style="width: 100%;"></el-date-picker>
                </el-col>
              </el-form-item>
              <el-form-item label="截止时间">
                <el-col :span="11">
                  <el-date-picker type="date" placeholder="选择日期" v-model="editTask.endTime" style="width: 100%;"></el-date-picker>
                </el-col>
              </el-form-item>
              <el-form-item label="重要程度">
                <el-select v-model="editTask.importance" placeholder="请选择任务重要程度">
                  <el-option label="普通任务" value="普通任务"></el-option>
                  <el-option label="重要任务" value="重要任务"></el-option>
                </el-select>
              </el-form-item>
              <el-form-item label="完成度">
                <el-slider v-model="editTask.completion"></el-slider>
              </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button @click="editDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="editTasks('editTasks')">确 定</el-button>
            </div>
          </el-dialog>
          <el-tab-pane label="已完成" width="200px">
            <el-table :data="finishedTask" style="width: 100%">
              <el-table-column label="任务内容" width="140" align="center">
                <template slot-scope="scope">
                  <span>{{scope.row.title}}</span>
                </template>
              </el-table-column>
              <el-table-column label="起始时间" width="150" align="center">
                <template slot-scope="scope">
                  <span>{{scope.row.startTime}}</span>
                </template>
              </el-table-column>
              <el-table-column label="截止时间" width="150" align="center">
                <template slot-scope="scope">
                  <span>{{scope.row.endTime}} </span>
                </template>
              </el-table-column>
              <el-table-column prop="importance" label="重要程度" width="120" align="center">
              </el-table-column>
              <el-table-column label="完成度" width="180" align="center">
                <template slot-scope="scope">
                  <div class="block" v-if="scope.row.completion!==100">
                    <el-slider v-model="scope.row.completion" :format-tooltip="formatTooltip" class="slider-diy"></el-slider>
                  </div>
                  <div class="completion_div" v-else-if="scope.row.completion===100">
                    <img src="./assets/check-circle.jpg" alt="uuu" />
                  </div>
                </template>
              </el-table-column>
              <el-table-column label="完成情况" width="150" align="center">
                <template slot-scope="scope">
                  <span>{{ getTaskDone(scope.row.completion) }}</span>
                </template>
              </el-table-column>
              <el-table-column prop="operator" width="240" align="center">
                <template slot-scope="scope">
                  <el-button type="danger" round @click="deleteFinishedTask(scope.$index, scope.row)">彻底删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
      </el-main>
    </el-container>
  </div>
</template>


<script>
import {formatDate} from './date'

export default {
  data() {
    return {
      startTime: [new Date(2020, 12, 10, 0, 0), new Date(2020, 12, 10, 0, 0)],
      date2: '',
      dialogFormVisible: false,
      index: 0,
      currentTasks: [{
        title: '吃饭',
        startTime: '2020-12-21',
        endTime: '2020-12-31',
        importance: '重要任务',
        completion: 50
      }, {
        title: '洗衣服',
        startTime: '2020-12-19',
        endTime: '2020-12-19',
        importance: '普通任务',
        completion: 30
      }, {
        title: '准备期末考',
        startTime: '2020-12-17',
        endTime: '2020-12-30',
        importance: '重要任务',
        completion: 10
      }, {
        title: '写vue实验',
        startTime: '2020-12-16',
        endTime: '2020-12-21',
        importance: '重要任务',
        completion: 0
      }],
      addDialogVisible: false,
      addTask: {
        title: '',
        startTime: '',
        endTime: '',
        priority: '',
        completion: 0,
      },

      editDialogVisible: false,
      editTask: {
        title: '',
        startTime: '',
        endTime: '',
        priority: '',
        completion: 0,
        content: '',
        desc: ''
      },

      finishedTask: [{
        title: '写计网',
        startTime: '2020-12-05',
        endTime: '2020-12-06',
        importance: '普通任务',
        completion: 100
      }, {
        title: '给饭卡充钱',
        startTime: '2020-12-12',
        endTime: '2020-12-12',
        importance: '重要任务',
        completion: 100
      }],
    }
  },

  filters: {
    formatDate(time) {
      let date = new Date(time);
      return formatDate(date, 'yyyy-MM-dd')
    }
  },

  computed: {
    getTaskDone() {
      return function (degree) {
        if (degree === 0) {
          return '待完成'
        } else if (degree <= 99) {
          return degree+"%"
        } else {
          return '已完成'
        }
      }
    }
  },
  methods: {

    getCurrentDate() {
      return formatDate(new Date(), 'yyyy-MM-dd')
    },
    tableColor(row) {
      if(row.column.label==="重要程度"){
        switch (row.row.importance){
          case "重要任务":return 'color: blue';
        }
      }
      if(row.column.label==="截止时间"){
        if(row.row.endTime<this.getCurrentDate()){
          return 'color:red';
        }
      }

      return '';
    },
    addTasks() {
      this.addDialogVisible = true;
    },

    confirm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.addDialogVisible = false;
          let resemble = JSON.parse(JSON.stringify(this.addTask));
          this.currentTasks.push(resemble)
        } else {
          console.log('错误！');
          return false;
        }
        this.addTask.title = "";
        this.addTask.region = "";
        this.addTask.startTime = null;
        this.addTask.endTime = null;
        this.addTask.priority = null;
        this.addTask.completion = 0;
        this.addTask.state = "";
        this.addTask.content = "";
        this.addTask.desc = "";
      });
    },

    edit(index, row) {
      console.log(index, row);
      this.editTask = JSON.parse(JSON.stringify(row));
      this.index = index;
      this.editDialogVisible = true;
    },

    editTasks(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.editDialogVisible = false;
          this.currentTasks.splice(this.index, 1, JSON.parse(JSON.stringify(this.editTask)));
          this.index = 0;
        } else {
          console.log('错误！');
          return false;
        }
        this.editDialogVisible = false
      });
    },

    deleteTask(index, row) {
      console.log(index, row);
      this.currentTasks.splice(index, 1);
    },

    deleteFinishedTask(index, row) {
      console.log(index, row);
      this.finishedTask.splice(index, 1);
    },

    finishTask(index, row) {
      this.finishedTask.push(JSON.parse(JSON.stringify(row)));
      this.currentTasks.splice(index, 1)
    },

    formatTooltip(val) {
      return val / 100;
    },

    filterHandler(value, row) {
      console.log(value);
      console.log(row['startTime']);
      return row['startTime'] === value;
    }
  }

}
</script>

<style>
\deep\ .slider-diy > .el-slider__button-wrapper .el-tooltip, .el-slider__button-wrapper .el-slider__button {
  width: 0;
  height: 0;
  border: 8px solid transparent;
  border-left-color: #409EFF;
  border-radius: 0;
  background: transparent;
  position: relative;
  left: 3px;
}


.completion_div > img {
  width: 20px;
  height: 20px;
}
</style>

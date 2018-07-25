<template>
  <div style="height: 100%">
    <el-popover ref="findBusyIdle" placement="bottom" width="18.301vw" v-model="isFindBusyIdle">
      <div id="find-busyIdle__dialog">
        <p>起止时间</p>
        <date-time-picker v-model="historyStartEnd"></date-time-picker>
        <div style="text-align: center">
          <el-button id="find__cancel" @click="isFindBusyIdle = false" type="text">取消</el-button>
          <el-button id="find__ensure" @click="findBusyIdle" type="text">确定</el-button>
        </div>
      </div>
    </el-popover>
    <div class="btns">
      <h3 class="timeTitle" v-html="timeTitle[0]+'至'+timeTitle[1]"></h3>
      <el-button id="find-busyIdle" class="btn" icon="el-icon-my-history" plain v-popover:findBusyIdle>选择日期</el-button>
    </div>
    <button v-show="showFlag" @click="backList" type="button" class="el-carousel__arrow el-carousel__arrow--left busyIdle__back"><i class="el-icon-arrow-left"></i></button>
    <div v-if="dataList.length == 0" id="no-evaluation-tips">
        暂无数据
    </div>
    <el-table v-show="!showFlag" :data="dataList" stripe class="table-style" @row-click="rowClick">
      <el-table-column prop="name" label="职位"></el-table-column>
      <el-table-column prop="avaTime" sortable label="可用工时"></el-table-column>
      <el-table-column prop="planTime" sortable label="计划工时"></el-table-column>
      <el-table-column prop="realTime" sortable label="实际工时"></el-table-column>
      <el-table-column prop="approval" sortable label="核准工时"></el-table-column>
      <el-table-column prop="busyTime" sortable label="忙闲工时"></el-table-column>
    </el-table>
    <div v-show="showFlag" style="margin-left: 4vw">
      <el-radio v-model="radio" border label="1">忙碌人员</el-radio>
      <el-radio v-model="radio" border label="2">闲置人员</el-radio>
    </div>
    <el-table v-show="showFlag" ref="table" :data="personList" stripe class="table-style" @row-click="toStatistics">
      <el-table-column prop="name" label="姓名"></el-table-column>
      <el-table-column prop="cityName" label="城市"></el-table-column>
      <el-table-column prop="jobName" label="岗位"></el-table-column>
      <el-table-column prop="avaTime" sortable label="可用工时"></el-table-column>
      <el-table-column prop="planTime" sortable label="计划工时"></el-table-column>
      <el-table-column prop="realTime" sortable label="实际工时"></el-table-column>
      <el-table-column prop="approval" sortable label="核准工时"></el-table-column>
      <el-table-column prop="busyTime" sortable label="忙闲工时"></el-table-column>
    </el-table>
  </div>
</template>
<script>
import { mapState, mapMutations } from 'vuex';
import { date } from '@/utils';
import UserList from '@/components/UserList';
import DateTimePicker from '@/components/DateTimePicker';
import UserAvatar from '@/components/UserAvatar';

const MON = date.getWeekStart();
const SUN = date.getWeekEnd();
const LAST_DATE = date.getLastDateOfMonth();

export default {
  name: 'busyIdle',
  components: {
    'user-avatar': UserAvatar,
    'date-time-picker': DateTimePicker,
  },
  data() {
    return {
      dataList: [],
      isFindBusyIdle: false,
      historyStartEnd: [],
      busyIdleHours: 0,
      personList: [],
      showFlag: false,
      timeTitle: [],
      jobId: 0,
      radio: '0',
      personListCopy: [],
    };
  },
  watch: {
    radio(val, oldVal) {
      this.personList = [];
      if (val === '1') {
        this.personListCopy.forEach((item, index) => {
          if (item.busyTime > 0) {
            this.personList.push(item);
          }
        });
      } else if (val === '2') {
        this.personListCopy.forEach((item, index) => {
          if (item.busyTime <= 0) {
            this.personList.push(item);
          }
        });
      }
    },
  },
  methods: {
    toStatistics(row, event, column) {
      this.$router.push({
        path: '/statistics',
        query: {
          startTime: this.historyStartEnd[0],
          endTime: this.historyStartEnd[1],
          userId: row.userId,
        },
      });
    },
    findBusyIdle() {
      let that = this;
      let params = {
        startTime: date.format(new Date(this.historyStartEnd[0]), 'yyyy-MM-dd'),
        endTime: date.format(new Date(this.historyStartEnd[1]), 'yyyy-MM-dd'),
        hours: this.busyIdleHours,
      };
      if (that.jobId === 0) {
        params.type = 2;
      } else {
        params.type = 3;
        params.id = this.jobId;
      }
      this.$api.$busyIdle.getBusyIdle(params, (res) => {
        if (that.jobId === 0) {
          that.dataList = res.retdata;
        } else {
          that.personList = res.retdata;
          that.personListCopy = res.retdata;
          that.showFlag = true;
        }
        that.timeTitle[0] = date.format(new Date(this.historyStartEnd[0]), 'yyyy-MM-dd');
        that.timeTitle[1] = date.format(new Date(this.historyStartEnd[1]), 'yyyy-MM-dd');
      });
      that.isFindBusyIdle = false;
    },
    backList() {
      this.showFlag = false;
      this.radio = '3';
      this.jobId = 0;
      this.findBusyIdle();
    },
    init() {
      let lastWeek = new Date();
      // lastWeek.setDate(lastWeek.getDate() - 7);
      this.historyStartEnd[0] = date.getWeekStart(lastWeek);
      this.historyStartEnd[1] = date.getWeekEnd(lastWeek);
      this.timeTitle[0] = date.format(new Date(this.historyStartEnd[0]), 'yyyy-MM-dd');
      this.timeTitle[1] = date.format(new Date(this.historyStartEnd[1]), 'yyyy-MM-dd');
    },
    valiDate(data) {
      let startTime = data[0];
      let endTime = data[1];
      if (!startTime || !endTime) {
        this.$message.error({ message: '请选择日期', center: true });
        return false;
      }
      startTime = new Date(startTime);
      endTime = new Date(endTime);
      return { startTime, endTime };
    },
    rowClick(row, event, column) {
      this.jobId = row.id;
      this.findBusyIdle();
    },
  },
  created() {
    this.init();
    this.findBusyIdle();
  },
};
</script>

<style lang="scss" scoped>
.table-style {
  width: 90%;
  margin:20px auto;
  max-height: 85%;
  overflow:scroll;
}
.table-style::-webkit-scrollbar {
  display: none;
}
.btns {
  width: 100%;
  overflow: hidden;
  #find-busyIdle {
    float: right;
    margin-right: 60px;
    &:hover,
    &:focus {
      color: $default;
      border-color: $default;
    }
  }
  .btn {
    @include setSize(110px, 52px);
    padding: 6px;
    font-size: 13px;
    vertical-align: super;
  }
}
#no-evaluation-tips {
  margin-top: 200px;
  text-align: center;
  color: $black;
  font-size: 32px;
}
#dusyIdle__table{
  width: 80%;
  height: 80%;
  margin: 20px auto 0 auto;
  overflow: scroll;
}

.busyIdle__back{
  border: none;
  outline: none;
  padding: 0;
  margin: 0;
  height: 2.635vw;
  width: 2.635vw;
  cursor: pointer;
  transition: .3s;
  border-radius: 50%;
  background-color: rgba(31, 45, 61, 0.11);
  color: #fff;
  position: absolute;
  top: 50%;
  left: 8vw;
  z-index: 10;
  transform: translateY(-50%);
  text-align: center;
  font-size: 0.878vw;
}
.timeTitle{
  float: left;
  color: #606266;
  font-weight: 400;
  margin: 0;
  margin-left: 4vw;
  padding: 0;
}
</style>

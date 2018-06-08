<template>
  <div style="height: 100%">
    <el-popover ref="findBusyIdle" placement="bottom" width="18.301vw" v-model="isFindBusyIdle">
      <div id="find-busyIdle__dialog">
        <p>起止时间</p>
        <date-time-picker v-model="historyStartEnd"></date-time-picker>
        <div style="text-align: center">
          <el-button id="find__cancel" @click="isFindHistory = false" type="text">取消</el-button>
          <el-button id="find__ensure" @click="findBusyIdle" type="text">确定</el-button>
        </div>
      </div>
    </el-popover>
    <div class="btns">
      <el-button id="find-busyIdle" class="btn" icon="el-icon-my-history" plain v-popover:findBusyIdle>选择日期</el-button>
    </div>
    <button v-show="showFlag" @click="showFlag = false" type="button" class="el-carousel__arrow el-carousel__arrow--left busyIdle__back"><i class="el-icon-arrow-left"></i></button>
    <div v-if="dataList.length == 0" id="no-evaluation-tips">
        暂无数据
    </div>
    <el-table v-show="!showFlag" :data="dataList" stripe style="width: 90%;margin:20px auto;" @row-click="rowClick">
      <el-table-column prop="name" label="职位"></el-table-column>
      <el-table-column prop="avaTime" label="可用工时"></el-table-column>
      <el-table-column prop="planTime" label="计划工时"></el-table-column>
      <el-table-column prop="realTime" label="实际工时"></el-table-column>
      <el-table-column prop="approval" label="核准工时"></el-table-column>
      <el-table-column prop="busyTime" label="忙闲工时"></el-table-column>
    </el-table>
    <el-table v-show="showFlag" :data="personList" stripe style="width: 90%;margin:20px auto;">
      <el-table-column prop="name" label="姓名"></el-table-column>
      <el-table-column prop="cityName" label="城市"></el-table-column>
      <el-table-column prop="jobName" label="岗位"></el-table-column>
      <el-table-column prop="avaTime" sortable label="可用工时"></el-table-column>
      <el-table-column prop="planTime" sortable label="计划工时"></el-table-column>
      <el-table-column prop="realTime" sortable label="实际工时"></el-table-column>
      <el-table-column prop="approval" sortable label="核准工时"></el-table-column>
      <el-table-column prop="busyTime" sortable label="忙闲工时" 
        :filters="[{ text: '闲置人员', value: '1' }, { text: '忙碌人员', value: '0' }]"
        :filter-method="filterTag"
        filter-placement="bottom-end">
      </el-table-column>
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
    };
  },
  methods: {
    findBusyIdle() {
      let that = this;
      let params = {
        startTime: date.format(new Date(this.historyStartEnd[0]), 'yyyy-MM-dd'),
        endTime: date.format(new Date(this.historyStartEnd[1]), 'yyyy-MM-dd'),
        hours: this.busyIdleHours,
        type: 2,
      };
      this.$api.$busyIdle.getBusyIdle(params, (res) => {
        that.dataList = res.retdata;
      });
    },
    init() {
      let lastWeek = new Date();
      // lastWeek.setDate(lastWeek.getDate() - 7);
      this.historyStartEnd[0] = date.getWeekStart(lastWeek);
      this.historyStartEnd[1] = date.getWeekEnd(lastWeek);
    },
    valiDate(data) {
      let startTime = data[0];
      let endTime = data[1];
      if (!startTime || !endTime) {
        this.$message.error({ message: '请选择日期', center: true });
        return false;
      }
      startTime = date.format(new Date(startTime), 'yyyy-MM-dd');
      endTime = date.format(new Date(endTime), 'yyyy-MM-dd');
      return { startTime, endTime };
    },
    rowClick(row, event, column) {
      let that = this;
      let params = {
        startTime: this.historyStartEnd[0],
        endTime: this.historyStartEnd[1],
        hours: this.busyIdleHours,
        type: 3,
        id: row.id,
      };
      this.$api.$busyIdle.getBusyIdle(params, (res) => {
        that.showFlag = true;
        that.personList = res.retdata;
      });
    },
    filterTag(value, row) {
      if (value === '1') {
        return row.busyTime > 0;
      }
      return row.busyTime <= 0;
    },
  },
  created() {
    this.init();
    this.findBusyIdle();
  },
};
</script>

<style lang="scss" scoped>
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
    @include setSize(110px, 32px);
    padding: 6px;
    font-size: 13px;
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
  margin: 20px auto;
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
</style>

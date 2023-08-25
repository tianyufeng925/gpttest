<template></template>
    <div class="cron-container">
      <el-form label-width="80px" ref="cronForm" :model="cronForm">
        <el-form-item label="周期">
          <el-radio-group v-model="cronForm.cycle">
            <el-radio label="每周"></el-radio>
            <el-radio label="每月"></el-radio>
            <el-radio label="每年"></el-radio>
            <el-radio label="指定日期"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="执行日期" v-if="cronForm.cycle === '每周'">
          <div v-for="(item, index) in cronForm.dayOfWeek" :key="index">
            <el-select v-model="item" placeholder="选择星期">
              <el-option
                v-for="(option, index) in weekOptions"
                :key="index"
                :label="option.label"
                :value="option.value"
              ></el-option>
            </el-select>
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="removeDayOfWeek(index)"
            ></el-button>
          </div>
          <el-button type="primary" icon="el-icon-plus" circle @click="addDayOfWeek"></el-button>
        </el-form-item>
        <el-form-item label="执行日期" v-if="cronForm.cycle === '每月'">
          <div v-for="(item, index) in cronForm.dayOfMonth" :key="index">
            <el-input-number
              v-model="item"
              :min="1"
              :max="31"
              placeholder="选择日期"
            ></el-input-number>
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="removeDayOfMonth(index)"
            ></el-button>
          </div>
          <el-button type="primary" icon="el-icon-plus" circle @click="addDayOfMonth"></el-button>
        </el-form-item>
        <el-form-item label="执行日期" v-if="cronForm.cycle === '每年'">
          <div v-for="(item, index) in cronForm.dateOfYear" :key="index">
            <el-date-picker
              v-model="item"
              type="month"
              placeholder="选择月份"
            ></el-date-picker>
            <el-input-number
              v-model.number="cronForm.dayOfMonth[index]"
              :min="1"
              :max="31"
              placeholder="选择日期"
            ></el-input-number>
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="removeDateOfYear(index)"
            ></el-button>
          </div>
          <el-button type="primary" icon="el-icon-plus" circle @click="addDateOfYear"></el-button>
        </el-form-item>
        <el-form-item label="执行日期" v-if="cronForm.cycle === '指定日期'">
          <div v-for="(item, index) in cronForm.date" :key="index">
            <el-date-picker
              v-model.number=item
              type=date
              placeholder=选择日期
            ></date-picker>
            <button
              type=danger
              icon=delete
              circle
              @click=removeDate(index)
            ></button>
          </div>
          <button type=primary icon=plus circle @click=addDate></button>
        </el-form-item>
        <el-form-item label="执行时间">
          <div v-for="(item, index) in cronForm.time" :key="index">
            <el-time-select
              v-model="item"
              :picker-options="{ start: '00:00', step: '00:15', end: '23:45' }"
            ></el-time-select>
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="removeTime(index)"
            ></el-button>
          </div>
          <el-button type="primary" icon="el-icon-plus" circle @click="addTime"></el-button>
        </el-form-item>
        <el-form-item label="Cron表达式">
          <el-input v-model="cronExpression" readonly></el-input>
        </el-form-item>
        <el-form-item label="最近5次运行时间">
          <ul>
            <li v-for="(time, index) in lastFiveTimes" :key="index">
              {{ time }}
            </li>
          </ul>
        </el-form-item>
      </el-form>
    </div>
  </template>
  
  <script>
import cronstrue from 'cronstrue' 
import cronParser from 'cron-parser' 
export default {
  data () {
    return {
      cronForm: {
        cycle: '每周',
        time: ['00:00'], 
        dayOfWeek: [], 
        dayOfMonth: [], 
        dateOfYear: [], 
        date: [] 
      },
      cronExpression: '',
      lastFiveTimes: [], 
      weekOptions: [

        { label: '星期一', value: 1 },
        { label: '星期二', value: 2 },
        { label: '星期三', value: 3 },
        { label: '星期四', value: 4 },
        { label: '星期五', value: 5 },
        { label: '星期六', value: 6 },
        { label: '星期日', value: 7 },
      ]
    }
  },
  watch: {
    cronForm: {
      handler () {
        this.generateCronExpression();
        this.getLastFiveTimes();
      },
      deep: true,
    }
  },
  methods: {
    generateCronExpression () {
      let minute = this.cronForm.time.map((item) => item.slice(3, 5)).join(',');
      let hour = this.cronForm.time.map((item) => item.slice(0, 2)).join(','); 
      let dayOfMonth = '*';
      let month = '*';
      let dayOfWeek = '*';
      switch (this.cronForm.cycle) {
        case '每周':
          if (this.cronForm.dayOfWeek.length > 0) {
            dayOfWeek = this.cronForm.dayOfWeek.join(',');
          } else {
            this.cronExpression = '';
            return;
          }
          break;
        case '每月':
          if (this.cronForm.dayOfMonth.length > 0) {
            dayOfMonth = this.cronForm.dayOfMonth.join(',');
          } else {
            this.cronExpression = '';
            return;
          }
          break;
        case '每年':
          if (this.cronForm.dateOfYear.length > 0 && this.cronForm.dayOfMonth.length > 0) {
            month = this.cronForm.dateOfYear.map((item) => new Date(item).getMonth() + 1).join(',');
            dayOfMonth = this.cronForm.dayOfMonth.join(',');
          } else {
            this.cronExpression = '';
            return;
          }
          break
        case '指定日期':
          if (this.cronForm.date.length > 0) {
            dayOfMonth = this.cronForm.date.map((item) => new Date(item).getDate()).join(','); 
            month = this.cronForm.date.map((item) => new Date(item).getMonth() + 1).join(',');
            dayOfWeek = '?';
          } else {
            this.cronExpression = '';
            return;
          }
          break;
      }
      this.cronExpression = `${minute} ${hour} ${dayOfMonth} ${month} ${dayOfWeek}`;
    },
    getLastFiveTimes () {
      if (this.cronExpression) {
        try {
  
          let nextMinute = new Date();
          nextMinute.setMinutes(nextMinute.getMinutes() + 1);
      
          let interval = cronParser.parseExpression(this.cronExpression, { currentDate: nextMinute, reverse: false });
          this.lastFiveTimes = [];
          for (let i = 0; i < 5; i++) {
            let time = interval.next().toString();

            time = new Date(time).toLocaleString();
            this.lastFiveTimes.push(time);
          }
        } catch (error) {
    
          this.lastFiveTimes = [];
        }
      } else {
  
        this.lastFiveTimes = [];
      }
    },
    addTime () {
      this.cronForm.time.push('00: 00');
    },
    removeTime (index) {
      this.cronForm.time.splice(index, 1);
    },
    addDayOfWeek () {
      this.cronForm.dayOfWeek.push(null);
    },
    removeDayOfWeek (index) {
      this.cronForm.dayOfWeek.splice(index, 1);
    },
    addDayOfMonth () {
      this.cronForm.dayOfMonth.push(null);
    },
    removeDayOfMonth (index) {
      this.cronForm.dayOfMonth.splice(index, 1);
    },
    addDateOfYear () {
      this.cronForm.dateOfYear.push(null);
      this.cronForm.dayOfMonth.push(null);
    },
    removeDateOfYear (index) {
      this.cronForm.dateOfYear.splice(index, 1);
      this.cronForm.dayOfMonth.splice(index, 1);
    },
    addDate () {
      this.cronForm.date.push(null);
    },
    removeDate (index) {
      this.cronForm.date.splice(index, 1);
    }
  }
}
 </script>
  <style scoped>
.cron-container {
  width: 500px;
  background-color: white;
}
</style>

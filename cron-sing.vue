<template>
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
      <!-- 根据周期的不同，显示不同的选项 -->
      <el-form-item label="执行日期" v-if="cronForm.cycle === '每周'">
        <el-select v-model="cronForm.dayOfWeek" placeholder="选择星期">
          <el-option
            v-for="(item, index) in weekOptions"
            :key="index"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="执行日期" v-if="cronForm.cycle === '每月'">
        <el-input-number
          v-model="cronForm.dayOfMonth"
          :min="1"
          :max="31" placeholder="选择日期"
        ></el-input-number>
      </el-form-item>
      <el-form-item label="执行日期" v-if="cronForm.cycle === '每年'">
        <el-date-picker
          v-model="cronForm.dateOfYear"
          type="month"
          placeholder="选择月份"
        ></el-date-picker>
        <el-input-number
          v-model="cronForm.dayOfMonth"
          :min="1"
          :max="31"
          placeholder="选择日期"
        ></el-input-number>
      </el-form-item>
      <el-form-item label="执行日期" v-if="cronForm.cycle === '指定日期'">
        <el-date-picker
          v-model="cronForm.date"
          type="date"
          placeholder="选择日期"
        ></el-date-picker>
      </el-form-item>
      <!-- 执行时间始终显示 -->
      <el-form-item label="执行时间">
        <el-time-select
          v-model="cronForm.time"
          :picker-options="{ start: '00:00', step: '00:15', end: '23:45' }"
        ></el-time-select>
      </el-form-item>
      <el-form-item label="Cron表达式">
        <el-input v-model="cronExpression" readonly></el-input>
      </el-form-item>
      <!-- 添加最近5次运行时间的显示 -->
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
import cronstrue from 'cronstrue' // 用于将cron表达式转换为自然语言
import cronParser from 'cron-parser' // 用于解析cron表达式并获取运行时间
export default {
  data () {
    return {
      cronForm: {
        cycle: '每周',
        time: '00:00',
        dayOfWeek: null,
        dayOfMonth: null,
        dateOfYear: null,
        date: null
      },
      cronExpression: '',
      lastFiveTimes: [], // 用于存储最近5次运行时间
      weekOptions: [ // 用于存储星期的选项
        { label: '星期一', value: 1 },
        { label: '星期二', value: 2 },
        { label: '星期三', value: 3 },
        { label: '星期四', value: 4 },
        { label: '星期五', value: 5 },
        { label: '星期六', value: 6 },
        { label: '星期日', value: 7 }
      ]
    }
  },
  watch: {
    cronForm: {
      handler () {
        this.generateCronExpression()
        this.getLastFiveTimes()
      },
      deep: true
    }
  },
  methods: {
    generateCronExpression () {
      let minute = this.cronForm.time.slice(3, 5)
      let hour = this.cronForm.time.slice(0, 2)
      let dayOfMonth = '*'
      let month = '*'
      let dayOfWeek = '*'
      switch (this.cronForm.cycle) {
        case '每周':
          if (this.cronForm.dayOfWeek) {
            dayOfWeek = this.cronForm.dayOfWeek
          } else {
            this.cronExpression = ''
            return
          }
          break
        case '每月':
          if (this.cronForm.dayOfMonth) {
            dayOfMonth = this.cronForm.dayOfMonth
          } else {
            this.cronExpression = ''
            return
          }
          break
        case '每年':
          if (this.cronForm.dateOfYear && this.cronForm.dayOfMonth) {
            let date = new Date(this.cronForm.dateOfYear)
            month = date.getMonth() + 1
            dayOfMonth = this.cronForm.dayOfMonth
          } else {
            this.cronExpression = ''
            return
          }
          break
        case '指定日期':
          if (this.cronForm.date) {
            let date = new Date(this.cronForm.date)
            dayOfMonth = date.getDate()
            month = date.getMonth() + 1
            dayOfWeek = '?'
          } else {
            this.cronExpression = ''
            return
          }
          break
      }
      this.cronExpression = `${minute} ${hour} ${dayOfMonth} ${month} ${dayOfWeek}`
        },
        getLastFiveTimes () {
  if (this.cronExpression) {
    try {
      // 获取当前时间的下一分钟
      let nextMinute = new Date()
      nextMinute.setMinutes(nextMinute.getMinutes() + 1)
      // 使用cron-parser库解析cron表达式，并获取当前时间之后的5个运行时间
      let interval = cronParser.parseExpression(this.cronExpression, { currentDate: nextMinute, reverse: false })
      this.lastFiveTimes = []
      for (let i = 0; i < 5; i++) {
        let time = interval.next().toString()
        // 使用toLocaleString方法来格式化时间
        time = new Date(time).toLocaleString()
        this.lastFiveTimes.push(time)
      }
    } catch (error) {
      // 如果cron表达式无效，清空最近5次运行时间的显示
      this.lastFiveTimes = []
    }
  } else {
    // 如果cron表达式为空，清空最近5次运行时间的显示
    this.lastFiveTimes = []
  }
}

    // getLastFiveTimes () {
    //   if (this.cronExpression) {
    //     try {
    //       // 使用cron-parser库解析cron表达式，并获取当前时间之前的5个运行时间
    //       let interval = cronParser.parseExpression(this.cronExpression, { currentDate: new Date(), reverse: true })
    //       this.lastFiveTimes = []
    //       for (let i = 0; i < 5; i++) {
    //         let time = interval.prev().toString()
    //         this.lastFiveTimes.push(time)
    //       }
    //     } catch (error) {
    //       // 如果cron表达式无效，清空最近5次运行时间的显示
    //       this.lastFiveTimes = []
    //     }
    //   } else {
    //     // 如果cron表达式为空，清空最近5次运行时间的显示
    //     this.lastFiveTimes = []
    //   }
    // }
  }
}
  </script>
  
  <style scoped>
.cron-container {
  width: 500px;
  background-color: white;
}
</style>
  
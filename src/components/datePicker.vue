<template>
  <div class="wy-date-picker">
    <input type="" :value="value" @focus="showTable = true" @blur="handleBlur">
    <div class="wy-date-table-wrap" v-show="showTable">
      <div>
        <button class="date-btn year-reduce" @click="yearReduce"></button>
        <button class="date-btn month-reduce" @click="monthReduce"></button>
        {{tmpYear}}年{{tmpMonth + 1}}月
        <button class="date-btn month-add" @click="monthAdd"></button>
        <button class="date-btn year-add" @click="yearAdd"></button>
      </div>
      <table cellspacing="0" cellpadding="0" class="date-table" @click="handleClick">
        <tbody>
          <tr>
            <th>日</th>
            <th>一</th>
            <th>二</th>
            <th>三</th>
            <th>四</th>
            <th>五</th>
            <th>六</th>
          </tr>
          <tr v-for="row in dateList">
            <td v-for="item in row">
              <div><span class="date-item" :class="{today: isToday(item), select: isSelectDay(item), previous: item.previousMonth, current: item.currentMonth, next: item.nextMonth, inRange: isInRange(item)}">{{item.value}}</span></div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
<script>
import dateFmt from '@/utils/dateFmt'
export default {
  name: 'wy-date-picker',
  props: [
    'value',
    'range'
  ],
  data () {
    return {
      dateValue: '',
      showTable: false,
      year: new Date().getFullYear(), // 选择的年
      month: new Date().getMonth(), // 选择的月
      day: new Date().getDate(),  // 选择的日
      tmpYear: new Date().getFullYear(), // 面板年
      tmpMonth: new Date().getMonth(),  // 面板月
      tmpStartYear: new Date().getFullYear(),
      tmpStartMonth: new Date().getMonth(),
      tmpStartDate: new Date().getDate(),
      tmpEndYear: new Date().getFullYear(),
      tmpEndMonth: new Date().getMonth(),
      tmpEndDate: new Date().getDate(),
      rangeStart: false
    }
  },
  created () {
    if (this.value && !this.range) {
      this.year = new Date(this.value).getFullYear()
      this.month = new Date(this.value).getMonth()
      this.day = new Date(this.value).getDate()
      this.tmpYear = new Date(this.value).getFullYear()
      this.tmpMonth = new Date(this.value).getMonth()
    } else if (this.value && this.range && (this.value instanceof Array)) {
      const dateStart = (this.value[0] instanceof Date) ? this.value[0] : new Date(this.value[0])
      const dateEnd = (this.value[1] instanceof Date) ? this.value[1] : new Date(this.value[1])
      if (dateStart.toString() === 'Invalid Date' || dateEnd.toString() === 'Invalid Date') {
        return false
      }
      this.tmpYear = dateStart.getFullYear()
      this.tmpMonth = dateStart.getMonth()
      this.tmpStartYear = dateStart.getFullYear()
      this.tmpStartMonth = dateStart.getMonth()
      this.tmpStartDate = dateStart.getDate()
      this.tmpEndYear = dateEnd.getFullYear()
      this.tmpEndMonth = dateEnd.getMonth()
      this.tmpEndDate = dateEnd.getDate()
    }
  },
  watch: {
    'value': function (val, oldVal) {
      if (val !== oldVal && !this.range) {
        this.year = new Date(this.value).getFullYear()
        this.month = new Date(this.value).getMonth()
        this.day = new Date(this.value).getDate()
        this.tmpYear = new Date(this.value).getFullYear()
        this.tmpMonth = new Date(this.value).getMonth()
      } else if (val !== oldVal && this.range) {
      }
    }
  },
  mounted () {
    document.addEventListener('click', this.close)
  },
  computed: {
    dateList () {
      let rows = [
        [],
        [],
        [],
        [],
        [],
        []
      ]
      let tmpYear = this.tmpYear || new Date().getFullYear()
      let tmpMonth = this.tmpMonth || new Date().getMonth()
      let currentMonthLength = new Date(tmpYear, tmpMonth + 1, 0).getDate()
      let dateList = Array.from({
        length: currentMonthLength
      }, (val, index) => {
        return {
          currentMonth: true,
          value: index + 1
        }
      })
      let startDay = new Date(tmpYear, tmpMonth, 1).getDay()
      let previousMongthLength = new Date(tmpYear, tmpMonth, 0).getDate()
      for (let i = 0, len = startDay; i < len; i++) {
        dateList = [{
          previousMonth: true,
          value: previousMongthLength - i
        }].concat(dateList)
      }
      for (let i = dateList.length, item = 1; i < 42; i++, item++) {
        dateList[dateList.length] = {
          nextMonth: true,
          value: item
        }
      }
      for (let i = 0; i < 6; i++) {
        rows[i] = dateList.splice(0, 7)
      }
      return rows
    },
    dateStart () {
      return +new Date(this.tmpStartYear, this.tmpStartMonth, this.tmpStartDate)
    },
    dateEnd () {
      return +new Date(this.tmpEndYear, this.tmpEndMonth, this.tmpEndDate)
    }
  },
  methods: {
    handleBlur (event) {
      if (!this.range) {
        const date = new Date(event.target.value)
        if (date.toString() === 'Invalid Date') {
          console.warn('请输入正确的日期')
        } else {
          this.$emit('input', dateFmt(date, 'yyyy-MM-dd'))
        }
      } else {

      }
    },
    isToday (item) {
      const year = new Date().getFullYear()
      const month = new Date().getMonth()
      const day = new Date().getDate()
      // 如果是当月
      if (item.currentMonth) {
        return year === this.tmpYear && month === this.tmpMonth && day === item.value
      }
      // 简单起见不在下个月显示当天的情况
    },
    isSelectDay (item) {
      // 单个日期选取
      if (!this.range && item.currentMonth) {
        return this.year === this.tmpYear && this.month === this.tmpMonth && this.day === item.value
      }
      // 范围日期选取(其实日期和结束日期都展示选中)
      if (this.range && item.currentMonth) {
        return (this.tmpStartYear === this.tmpYear && this.tmpStartMonth === this.tmpMonth && this.tmpStartDate === item.value) || (this.tmpEndYear === this.tmpYear && this.tmpEndMonth === this.tmpMonth && this.tmpEndDate === item.value)
      }
    },
    isInRange (item) {
      const {currentYear, currentMonth} = this.calDate(item)
      if (!this.range) {
        return false
      } else {
        // 区别current prev next
        const dateItem = +new Date(currentYear, currentMonth, item.value)
        return dateItem > this.dateStart && dateItem < this.dateEnd
      }
    },
    handleClick (event) {
      let target = event.target
      if (target.tagName === 'SPAN') {
        target = target.parentNode.parentNode
      }
      if (target.tagName === 'DIV') {
        target = target.parentNode
      }
      if (target.tagName !== 'TD') return
      const cellIndex = target.cellIndex
      const rowIndex = target.parentNode.rowIndex - 1
      const select = this.dateList[rowIndex][cellIndex]
      const {currentYear, currentMonth} = this.calDate(select)
      this.tmpYear = currentYear
      this.tmpMonth = currentMonth
      // 非范围选中情况
      if (!this.range) {
        this.year = currentYear
        this.month = currentMonth
        this.day = select.value
        const dateStr = dateFmt(new Date(this.year, this.month, this.day), 'yyyy-MM-dd')
        this.$emit('input', dateStr)
        this.$emit('change', dateStr)
        this.showTable = false
      } else if (this.range && !this.rangeStart) {
        this.tmpStartYear = this.tmpEndYear = currentYear
        this.tmpStartMonth = this.tmpEndMonth = currentMonth
        this.tmpStartDate = this.tmpEndDate = select.value
        this.rangeStart = true
      } else if (this.range && this.rangeStart) {
        this.tmpEndYear = currentYear
        this.tmpEndMonth = currentMonth
        this.tmpEndDate = select.value
        this.rangeStart = false

        // 先选择截至日期在选择开始日期
        if (this.dateStart > this.dateEnd) {
          let dateStart = new Date(this.dateEnd)
          let dateEnd = new Date(this.dateStart)
          this.tmpStartYear = dateStart.getFullYear()
          this.tmpStartMonth = dateStart.getMonth()
          this.tmpStartDate = dateStart.getDate()
          this.tmpEndYear = dateEnd.getFullYear()
          this.tmpEndMonth = dateEnd.getMonth()
          this.tmpEndDate = dateEnd.getDate()
        }
        const dateStartStr = dateFmt(new Date(this.dateStart), 'yyyy-MM-dd')
        const dateEndStr = dateFmt(new Date(this.dateEnd), 'yyyy-MM-dd')
        this.$emit('input', [dateStartStr, dateEndStr])
        this.$emit('change', [dateStartStr, dateEndStr])
      }
    },
    calDate (item) {
      let currentYear = this.tmpYear
      let currentMonth = this.tmpMonth
      if (item.currentMonth) {
      } else {
        if (item.previousMonth) {
          currentMonth -= 1
          if (currentMonth < 0) {
            currentYear -= 1
            currentMonth = 11
          }
        }
        if (item.nextMonth) {
          currentMonth += 1
          if (currentMonth > 11) {
            currentYear += 1
            currentMonth = 0
          }
        }
      }
      return {
        currentYear,
        currentMonth
      }
    },
    yearReduce () {
      if (this.tmpYear > 1970) {
        this.tmpYear -= 1
      }
    },
    yearAdd () {
      if (this.tmpYear < 2050) {
        this.tmpYear += 1
      }
    },
    monthAdd () {
      this.tmpMonth += 1
      if (this.tmpMonth > 11) {
        this.yearAdd()
        this.tmpMonth = 0
      }
    },
    monthReduce () {
      this.tmpMonth -= 1
      if (this.tmpMonth < 0) {
        this.yearReduce()
        this.tmpMonth = 11
      }
    },
    close (e) {
      if (!this.$el.contains(e.target)) {
        this.showTable = false
      }
    }
  },
  destory () {
    document.removeEventListener('click', this.close)
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wy-date-picker {
  position: relative;
  width: 200px;
  height: 40px;
  line-height: 40px;
  font-size: 12px;
  text-align: left;
}

.wy-date-table-wrap {
  z-index: 5000;
  position: absolute;
  top: 40px;
  left: 0;
  width: 322px;
  padding: 10px 20px;
  line-height: 1.2;
  text-align: center;
  border: 1px solid #eee;
  box-shadow: 5px 5px 5px #ccc;
}

.date-table {
  table-layout: fixed;
  width: 100%;
}

.date-btn{
  border: 1px solid #ccc;
  background: none;
  cursor: pointer;
}

.year-add:before {
  content: ">>";
}

.year-reduce:before {
  content: "<<";
}

.month-add:before {
  content: ">";
}

.month-reduce:before {
  content: "<";
}

.today{
  color: red;
}

.date-item{
  position: absolute;
  cursor: pointer;
  display: block;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 24px;
  height: 24px;
  line-height: 24px;
}

.date-item:hover{
  color: #409EFF;
}

.select{
  border-radius: 50%;
  background: #409EFF;
  color: #FFF!important;
}

.inRange{
  background: #eee;
}

.previous, .next{
  color: #ccc;
}

th {
  padding: 5px;
  border-bottom: 1px solid #ccc
}

td {
  position: relative;
  width: 40px;
  height: 30px;
  padding: 0 5px;
  line-height: 30px;
}
</style>

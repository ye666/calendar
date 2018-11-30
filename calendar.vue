<template>
  <div class="calendar-container">
    <div class="year">
      <div>
        <span class="fl"
              @click="lastYear">«
        </span>
        <span class="fl"
              @click="lastMonth">‹</span>
        <p>{{nowDate.year}}年{{nowDate.month+1}}月</p>
        <span class="fr"
              @click="nextYear">»</span>
        <span class="fr"
              @click="nextMonth">›</span>
      </div>
    </div>
    <ul class="week">
      <li v-for="(o,index) in 7"
          :key="o">{{formatWeek(index)}}</li>
    </ul>
    <ul class="date">
      <li class="none-week"
          v-for="o in lastMonthDays"
          :key="o+50">{{lastMonthStartDay+o-1}}</li>
      <li @click="clickEvent(day.day)"
          v-for="day in getday1"
          :class="{'nowday':day.day===clickday}"
          :key="day.day">
        <div style="margin-top:3px">{{day.day}}</div>
        <div class="dot"
             v-show="day.show"
             :class="{'dot1':day.day===clickday}"></div>
      </li>
      <li class="none-week"
          v-for="day in (42-lastMonthDays-nowMonthDays)"
          :key="day+100">{{day}}</li>
    </ul>
  </div>
</template>

<script>
export default {
  // props: ["start-date"],
  data() {
    return {
      selectDate: [], //选择日期列表
      nowDate: this.getDate(new Date()), //当前设置时间 默认为当前系统时间
      clickday: "",
      getday1: [],
      locusday: [],
      startTime: "",
      endTime: "",
      adminUuid: sessionStorage.getItem("adminUuid")
    };
  },
  computed: {
    lastMonthDays() {
      return this.startWeek();
    },
    lastMonthStartDay() {
      return (
        this.calcLastMonthDays(this.nowDate.year, this.nowDate.month) -
        (this.startWeek() - 1)
      );
    },
    nowMonthDays() {
      return this.calcDays(this.nowDate.year, this.nowDate.month);
      // this.getday1=this.calcDays(this.nowDate.year, this.nowDate.month);
      // console.log(this.calcDays(this.nowDate.year, this.nowDate.month))
      // console.log('asd')
    }
  },
  watch: {
    nowMonthDays() {
      // console.log(this.nowMonthDays)
      // this.getday1=this.nowMonthDays
      this.nowMonthDate();
    },
    startTime() {
      // console.log(this.startTime)
      this.work_locus();
      // console.log(this.startTime)
      // console.log(this.endTime)
    }
  },
  created() {
    if (this.setDate) {
      this.nowDate = this.getDate(this.setDate);
    }
    // this.getday1=this.nowMonthDays
    this.clickday = this.nowDate.date;
    this.nowMonthDate();
    this.locusTime();
    // this.work_locus()
    // console.log(this.nowMonthDays);
  },
  methods: {
    getDate(date) {
      return {
        year: date.getFullYear(),
        month: date.getMonth(),
        day: date.getDay(),
        date: date.getDate()
      };
    },

    nowMonthDate() {
      // console.log(this.getday1)
      this.getday1 = [];
      let i = 1;
      for (i = 1; i <= this.nowMonthDays; i++) {
        this.getday1.push({ day: i, show: false });
      }
      // console.log(this.getday1);
    },
    formatWeek(day) {
      switch (day) {
        case 0:
          return "日";
        case 1:
          return "一";
        case 2:
          return "二";
        case 3:
          return "三";
        case 4:
          return "四";
        case 5:
          return "五";
        case 6:
          return "六";
      }
    },
    //判断闰年
    isLeapYear(year) {
      return year % 4 == 0 && (year % 100 != 0 || year % 400 == 0);
    },
    //根据日子计算星期
    calcWeekend(year, month, day) {
      return new Date(year, month, day).getDay();
    },
    //计算某年某月的天数
    calcDays(year, month) {
      const monthDay = [
        { days: 31 },
        { days: 28 },
        { days: 31 },
        { days: 30 },
        { days: 31 },
        { days: 30 },
        { days: 31 },
        { days: 31 },
        { days: 30 },
        { days: 31 },
        { days: 30 },
        { days: 31 }
      ];
      if (this.isLeapYear(year) && month === 1) return 29;
      else return monthDay[month].days;
      // console.log(monthDay[month].days)
    },
    //计算上个月天数
    calcLastMonthDays(year, month) {
      if (month === 0) {
        return this.calcDays(year - 1, 11);
      } else {
        return this.calcDays(year, month - 1);
      }
    },
    //上月
    lastMonth() {
      if (this.nowDate.month === 0) {
        this.nowDate.month = 11;
        this.nowDate.year--;
      } else {
        this.nowDate.month--;
      }
      //设置工作轨迹的开始时间和结束时间
      this.startTime =
        this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "1";
      this.endTime =
        this.nowDate.year +
        "-" +
        (this.nowDate.month + 1) +
        "-" +
        this.nowMonthDays;
      let monthNo = this.nowDate.month;
      let month = monthNo <= 11 ? monthNo + 1 : 0;
      let date = {
        year: this.nowDate.year,
        month: month,
        week: new Date(this.nowDate.year, this.nowDate.month, 1).getDay(),
        day: 1
      };
      this.clickday = date.day;
      this.$emit("click-event", date);
      // console.log(this.nowMonthDays)
      // this.getday1=this.nowMonthDays
    },
    //下月
    nextMonth() {
      if (this.nowDate.month === 11) {
        this.nowDate.month = 0;
        this.nowDate.year++;
      } else {
        this.nowDate.month++;
      }
      //设置工作轨迹的开始时间和结束时间

      this.startTime =
        this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "1";
      this.endTime =
        this.nowDate.year +
        "-" +
        (this.nowDate.month + 1) +
        "-" +
        this.nowMonthDays;
      let monthNo = this.nowDate.month;
      let month = monthNo <= 11 ? monthNo + 1 : 0;
      let date = {
        year: this.nowDate.year,
        month: month,
        week: new Date(this.nowDate.year, this.nowDate.month, 1).getDay(),
        day: 1
      };
      this.clickday = date.day;
      this.$emit("click-event", date);
    },
    //去年
    lastYear() {
      this.nowDate.year--;
      //设置工作轨迹的开始时间和结束时间
      // let time1 =
      //   this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "1";
      // let time2 = "";
      // let time3 = "";
      // if (this.nowDate.month != 11) {
      //   time2 = this.nowDate.year + "-" + (this.nowDate.month + 2) + "-" + "1";
      //   time3 = new Date(new Date(time2).getTime() - 24 * 60 * 60 * 1000);
      // } else {
      //   time2 = this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "31";
      //   time3 = new Date(new Date(time2).getTime());
      // }

      // // console.log(time3);
      // this.startTime = Date.parse(time1);
      // this.endTime = Date.parse(time3);
      // console.log(this.endTime)
      this.startTime =
        this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "1";
      this.endTime =
        this.nowDate.year +
        "-" +
        (this.nowDate.month + 1) +
        "-" +
        this.nowMonthDays;
      let monthNo = this.nowDate.month;
      let month = monthNo <= 11 ? monthNo + 1 : 0;
      let date = {
        year: this.nowDate.year,
        month: month,
        week: new Date(this.nowDate.year, this.nowDate.month, 1).getDay(),
        day: 1
      };
      this.clickday = date.day;
      this.$emit("click-event", date);
    },
    //下一年
    nextYear() {
      this.nowDate.year++;

      this.startTime =
        this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "1";
      this.endTime =
        this.nowDate.year +
        "-" +
        (this.nowDate.month + 1) +
        "-" +
        this.nowMonthDays;
      let monthNo = this.nowDate.month;
      let month = monthNo <= 11 ? monthNo + 1 : 0;
      let date = {
        year: this.nowDate.year,
        month: month,
        week: new Date(this.nowDate.year, this.nowDate.month, 1).getDay(),
        day: 1
      };
      this.clickday = date.day;
      this.$emit("click-event", date);
      // console.log(this.endTime)
    },
    //计算当月开始星期
    startWeek() {
      return this.calcWeekend(this.nowDate.year, this.nowDate.month, 1);
    },
    locusTime() {
      this.startTime =
        this.nowDate.year + "-" + (this.nowDate.month + 1) + "-" + "1";

      this.endTime =
        this.nowDate.year +
        "-" +
        (this.nowDate.month + 1) +
        "-" +
        this.nowMonthDays;

      // console.log(this.endTime)
    },
    work_locus() {
      this.$ajax("work_locus", {
        adminUuid: this.adminUuid,
        startDate: this.startTime,
        endDate: this.endTime
      }).then(e => {
        if (e.data.code === 1) {
          // this.$dialog.toast({
          //     mes: e.data.data,
          //     timeout: 5000
          //   });
          this.nowMonthDate();
          for (let i in e.data.data) {
            // this.$dialog.toast({
            //   mes: new Date(e.data.data[i]).getDate(),
            //   timeout: 5000
            // });
            for (let j in this.getday1) {
              if (new Date(e.data.data[i]).getDate() === this.getday1[j].day) {
                this.getday1[j].show = true;
              }
            }
          }
        }
        // this.$dialog.toast({
        //   mes: e.data,
        //   timeout: 5000
        // });
      });
    },
    //
    clickEvent(e) {
      // this.$dialog.toast({
      //   mes: e,
      //   timeout: 5000
      // });
      let monthNo = this.nowDate.month;
      let month = monthNo <= 11 ? monthNo + 1 : 0;
      let date = {
        year: this.nowDate.year,
        month: month,
        week: new Date(this.nowDate.year, this.nowDate.month, e).getDay(),
        day: e
      };
      this.clickday = date.day;
      this.$emit("click-event", date);
    }
  }
};
</script>


<style scoped>
.nowday {
  background: #177ef3;
  color: #ffffff;
  border-radius: 4px;
}
.year {
  text-align: center;
  margin-bottom: 10px;
  height: 30px;
}
.week,
.date {
  box-sizing: border-box;
  display: flex;
  flex-wrap: wrap;
  list-style: none;
}
.week {
  /* border-bottom: 0.5px solid #ddd; */
  margin-bottom: 5px;
}
ul > li {
  font-size: 14px;
  /* width: calc(94vw / 7); */
  width: 14%;
  height: 30px;
  text-align: center;
  /* line-height: 30px; */
}
.dot {
  margin: auto;
  width: 4px;
  height: 4px;
  border-radius: 90px;
  background: #177ef3;
}
.dot1 {
  margin: auto;
  width: 4px;
  height: 4px;
  border-radius: 90px;
  background: #ffffff;
}
.none-week {
  color: #adaeb0;
}
.year > div {
  height: 30px;
  overflow: hidden;
}
.year span {
  line-height: 30px;
  font-size: 20px;
  display: inline-block;
  width: 10%;
}
.year p {
  line-height: 2;
  width: 50%;
  display: inline-block;
}

</style>
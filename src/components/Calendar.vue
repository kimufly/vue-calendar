<template>
  <div>
  <div class="calendar-box" :class="{'calendar-show': toggle,'calendar-hide': !toggle}">
    <div class="calendar-bar">
      <div class="calendar-bars calendar-year-bar">
        <div class="calendar-bar-btn calendar-bar-btn-prev" @click="calendarBtn(0)"><i></i></div>
        <div class="calendar-current-value">{{ year }}</div>
        <div class="calendar-bar-btn calendar-bar-btn-next" @click="calendarBtn(1)"><i></i></div>
      </div>
      <div class="calendar-bars calendar-month-bar">
        <div class="calendar-bar-btn calendar-bar-btn-prev" @click="calendarBtn(2)"><i></i></div>
        <div class="calendar-current-value">{{month | month_zh }}月</div>
        <div class="calendar-bar-btn calendar-bar-btn-next" @click="calendarBtn(3)"><i></i></div>
      </div>      
    </div>
    <div class="calendar-inner">
      <ul class="calendar-week-days">
        <li>周日</li>
        <li>周一</li>
        <li>周二</li>
        <li>周三</li>
        <li>周四</li>
        <li>周五</li>
        <li>周六</li>
      </ul>
      <div class="calendar-months">
        <div class="calendar-months-wrap" 
          :style="{transform: transform_wrap}" 
          @touchstart="touchstartX"
          @touchmove="touchmoveX"
          @touchend="touchendX"
          ref="root"
        >       
          <div class="calendar-month calendar-month-prev" :style="{transform: transform_prev }">
            <div class="calendar-row" v-for="prevMonth in prevMonth">
              <div class="calendar-day" v-for="lineMonth in prevMonth"><span>{{ lineMonth.day }}</span></div>
            </div>            
          </div>
          <div class="calendar-month calendar-month-current" :style="{transform: transform_current }">
            <div class="calendar-row" v-for="curMonth in curMonth">
              <div class="calendar-day" 
                v-for="lineMonth in curMonth" 
                :class="{'calendar-day-ash': lineMonth.month != month, 
                          'calendar-day-today': val == '' ? lineMonth.date == today : (lineMonth.date != val && lineMonth.date == today), 
                          'calendar-day-selected': val == '' ? false : lineMonth.date == val
                        }" 
                :data-year="lineMonth.year" :data-month="lineMonth.month" :data-day="lineMonth.day" :data-date="lineMonth.date"
                @click="selectDate(lineMonth.date,lineMonth.month == month)"
              >
                <span>{{ lineMonth.day }}</span>
              </div>
            </div>
          </div>
          <div class="calendar-month calendar-month-next" :style="{transform: transform_next }">
            <div class="calendar-row" v-for="nextMonth in nextMonth">
              <div class="calendar-day" v-for="lineMonth in nextMonth"><span>{{ lineMonth.day }}</span></div>
            </div>            
          </div>
        </div>               
      </div>
    </div>
  </div>
  <div class="mask" :class="{'mask-show': toggle,'mask-hide': !toggle}" @click="closeCalender"></div>
  </div>
</template>

<script>
let iStartX = 0; //记录手指开始按下的X坐标
let iStartTranslateX = 0; //物体移动到的X坐标
let transform = 0; // 盒子初始偏移值

export default {
  name: 'calendar',
  data () {
    return {
      toggle: true, // 组件显示动画
      val: this.toval, // 初始化选中日期
      today: '', // 当天日期
      prevMonth: [], // 上月数据
      curMonth: [], // 本月数据
      nextMonth: [], // 下月数据
      year: '', // 年份
      month: '', // 月份
      day: '', // 今日日期
      transform_arr: [0,-100,0,100], // 盒子偏移基础数据
      transform_wrap: '',
      transform_prev: '',
      transform_current: '',
      transform_next: ''     
    }
  },
  props: ['toval'],
  mounted () {     
    this.$nextTick(()=>{
      let date = new Date();
      // 如果初始化日期没有设置，使用当天日期
      if(this.val == ''){
        this.year = Number(date.getFullYear());
        this.month = Number(date.getMonth() + 1);
        this.day = Number(date.getDate());  
      }else{
        let _val = this.val.split('-');
        this.year = Number(_val[0]);
        this.month = Number(_val[1]);
        this.day = Number(_val[2]);
      }
      this.today = date.getFullYear() + '-' + (date.getMonth() + 1) + '-' + date.getDate();
      this.init();
    });
  },
  filters: {
    month_zh (value){ // 中文月数
      let month_zh_arr = ['一','二','三','四','五','六','七','八','九','十','十一','十二'];
      return month_zh_arr[value - 1];
    }
  },
  methods: {
    init (){
      this.prevMonth = this.setMonth_arr(this.month - 1);
      this.curMonth = this.setMonth_arr(this.month);
      this.nextMonth = this.setMonth_arr(this.month + 1);
      this.transform_wrap = this.transform_str(this.transform_arr[0]);
      this.transform_prev = this.transform_str(this.transform_arr[1]);
      this.transform_current = this.transform_str(this.transform_arr[2]);
      this.transform_next = this.transform_str(this.transform_arr[3]);
    },
    transform_str (value){
      return 'translate3d(' + value + '%,0,0)';
    },
    transform_fun (direction){
      if(direction == 'left'){
        this.transform_arr[0] = this.transform_arr[0] + 100;
        this.transform_arr[1] = this.transform_arr[1] - 100;
        this.transform_arr[2] = this.transform_arr[2] - 100;
        this.transform_arr[3] = this.transform_arr[3] - 100;        
      }else if(direction == 'right'){
        this.transform_arr[0] = this.transform_arr[0] - 100;
        this.transform_arr[1] = this.transform_arr[1] + 100;
        this.transform_arr[2] = this.transform_arr[2] + 100;
        this.transform_arr[3] = this.transform_arr[3] + 100;
      }
    },    
    is_leap (year) {  // 是否为闰年
      return (year % 100 == 0 ? (year % 400 == 0 ? 1 : 0) : (year % 4 == 0 ? 1 : 0));
    },
    day_arr (year,month) {  // 各月份的总天数
      let day_str = new Array(31,28 + this.is_leap(year),31,30,31,30,31,31,30,31,30,31);
      return day_str[month - 1];
    },
    firstday (year,month) { // 当月第一天星期几
      return new Date(year, month - 1, 1).getDay();
    },
    setMonth_arr (month) { // 重置本月数据
      let idx;     
      let _year;
      let _month;
      let _day;
      let year = this.year;
      let month_line = [];
      let month_arr = [];

      if(month < 1){
        month = 12;
        year = this.year - 1;
      }else if(month > 12){
        month = 1;
        year = this.year + 1;
      }

      // i => 行 k => 列，按照日历布局格式输出月数据数组
      for(let i = 0; i < 6; i++) {
        month_line = [];        
        for(let k = 0; k < 7; k++) {
          idx = i * 7 + k;
          _day = idx - this.firstday(year,month) + 1; // 计算日期 
          _year = year;
          _month = month;         
          if(_day > this.day_arr(year,month)){            
            if(month + 1 > 12){
              _year = year + 1;
              _month = 1;
            }else{
              _month = month + 1;
            }
            _day = idx - this.firstday(year,month) + 1 - this.day_arr(year,month);
          }else if(_day <= 0){           
            if(month - 1 < 1){
              _month = 12;
              _year = year - 1;
            }else{
              _month = month - 1;             
            }
            _day = this.day_arr(_year,_month) + (idx - this.firstday(year,month) + 1);
          }
          month_line.push({
            "year": _year,
            "month": _month,
            "day": _day,
            "date": _year + '-' + _month + '-' + _day
          });
        }
        month_arr.push(month_line);
      }
      return month_arr;
    },
    calendarBtn (i){
      // i = 0 => 上一年，i = 1 => 下一年，i = 2 => 上一月，i = 3 => 下一月
      // 最小到1970年1月
      switch (i){
        case 0:
          this.year == 1970 ? this.year = this.year : this.year = this.year - 1;
          this.transform_fun('left');
          break;
        case 1:
          this.year = this.year + 1;
          this.transform_fun('right');
          break;
        case 2:
          if(this.year == 1970 && this.month == 1){
            this.month = this.month;
          }else if(this.year == 1970 && this.month > 1){
            this.month = this.month - 1;
          }else if(this.year > 1970 && this.month == 1){
            this.month = 12;
            this.year = this.year - 1;
          }else if(this.year > 1970 && this.month > 1){
            this.month = this.month - 1;
          }
          this.transform_fun('left');
          break;
        case 3:
          if(this.month == 12){
            this.month = 1;
            this.year = this.year + 1;
          }else{
            this.month = this.month + 1;
          }
          this.transform_fun('right');
          break;                    
      }
      this.init(); 
    },
    selectDate (date,flag){
      // date => 选中日期 yy-mm-dd, flag判断日期是否为当月日期，为false时不执行点击事件,反之执行点击事件，回调选中日期并关闭组件
      if(!flag){ return }
      this.toggle = !this.toggle;       
      this.$emit('dateVal', date);        
    },
    touchstartX (){
      const touchOne = event.changedTouches[0];
      iStartX = touchOne.pageX;
      this.$refs.root.style.transition = '0ms';
      transform = Number(this.$refs.root.style.WebkitTransform.match(/translate3d\((\S*)%/)[1]);
      iStartTranslateX = transform / 100 * this.$refs.root.offsetWidth;
    },
    touchmoveX (){
      const iMoveX = event.changedTouches[0].pageX - iStartX;
      this.$refs.root.style.WebkitTransform = this.$refs.root.style.transform = 'translate3d(' + (iStartTranslateX + iMoveX) + 'px,' + '0,0)';       
    },
    touchendX (){
      let iMoveX = event.changedTouches[0].pageX - iStartX;
      this.$refs.root.style.transition = '300ms';
      if( (iMoveX < 0 && iMoveX < -100) || (iMoveX > 0 && iMoveX > 100) ){
        if( iMoveX < 0){
          this.calendarBtn(3);
        }else{
          this.calendarBtn(2);
        }
      }else{
        this.$refs.root.style.WebkitTransform = this.$refs.root.style.transform = 'translate3d(' + transform + '%,' + '0,0)';       
      }      
    }    
  }
}
</script>

<style scoped>
*{margin: 0;padding: 0;}
ul,li{list-style-type: none;}
.calendar-box{width: 100%;height: 300px;overflow: hidden;background: #fff;position: fixed;left: 0;bottom: 0;z-index: 999;border-top: 1px solid #ccc;}

.calendar-box.calendar-show{
  -webkit-transform: translate3d(0,0,0);transform: translate3d(0,0,0);
  -webkit-animation: calendarShow .3s ease;animation: calendarShow .3s ease;
}
@-webkit-keyframes calendarShow{
  0%{-webkit-transform: translate3d(0,100%,0);}
  100%{-webkit-transform: translate3d(0,0,0);} 
}  
@keyframes calendarShow{
  0%{transform: translate3d(0,100%,0);}
  100%{transform: translate3d(0,0,0);} 
}

.calendar-box.calendar-hide{
  -webkit-transform: translate3d(0,100%,0);transform: translate3d(0,100%,0);
  -webkit-animation: calendarHide .3s ease;animation: calendarHide .3s ease;
}
@-webkit-keyframes calendarHide{
  0%{-webkit-transform: translate3d(0,0,0);}
  100%{-webkit-transform: translate3d(0,100%,0);} 
}  
@keyframes calendarHide{
  0%{transform: translate3d(0,0,0);}
  100%{transform: translate3d(0,100%,0);} 
}

.mask-show{display: block;}

.calendar-bar{height: 44px;}
  .calendar-bar .calendar-bars{float: left;display: -webkit-box;width: 50%;height: 100%;}
  .calendar-bar .calendar-bars .calendar-bar-btn{display: inline-block;width: 44px;height: 44px;}
  .calendar-bar .calendar-bars .calendar-current-value{-webkit-box-flex: 1;text-align: center;line-height: 44px;}
  .calendar-bar .calendar-bars .calendar-bar-btn i{display: block;width: 0;height: 0;border-top: 10px solid transparent;border-bottom: 10px solid transparent;margin: 11px;}
  .calendar-bar .calendar-bars .calendar-bar-btn-prev i{border-right: 20px solid #007AFF;}
  .calendar-bar .calendar-bars .calendar-bar-btn-next i{border-left: 20px solid #007AFF;}

.calendar-inner{height: 256px;}

.calendar-week-days{height: 20px;display: -webkit-box;background: #f7f7f7;}
  .calendar-week-days li{-webkit-box-flex: 1;text-align: center;line-height: 20px;font-size: 14px;}

.calendar-months{height: 236px;overflow: hidden;}
  .calendar-months-wrap{height: 100%;position: relative;-webkit-transition: 300ms;transition: 300ms;}
  .calendar-month{position: absolute;top: 0;bottom: 0;left: 0;right: 0;}
  .calendar-row{display: -webkit-box;height: 38px;border-top: 1px solid #CCCCCC;}
  .calendar-row .calendar-day{-webkit-box-flex: 1;}
  .calendar-row .calendar-day span{display: block;width: 30px;line-height: 30px;margin: 4px auto;text-align: center;font-size: 14px;color: #3d4145}
  .calendar-row .calendar-day.calendar-day-ash span{color: #ccc;}
  .calendar-row .calendar-day.calendar-day-selected span{color: #fff;background: #007AFF;border-radius: 50%;}
  .calendar-row .calendar-day.calendar-day-today span{background: #e3e3e3;border-radius: 50%;}
</style>
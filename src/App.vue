<template>
  <div id="app">
    <input type="" name="" :value="to_val" @click="toggleCalendar = true">
    <keep-alive v-if="toggleCalendar">
      <calendar :toval="to_val" @dateVal="date_Val"></calendar>
    </keep-alive>
  </div>
</template>

<script>
import calendar from './components/Calendar'

export default {
  name: 'app',
  data (){
    return {
      toggleCalendar: false,
      to_val: '2017-4-26'        
    }
  },  
  components: { calendar },
  methods: {
    date_Val (date){
      this.to_val = date;
      setTimeout(()=>{
        this.toggleCalendar = !this.toggleCalendar;
      },300);
    }
  }    
}
</script>

<style>

</style>
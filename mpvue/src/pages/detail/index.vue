<template>
  <div class="counter">
    <swiper :display-multiple-items="displaymultipleitems" :interval="interval" :duration="duration" :current='(clickindex<forecastlist.length-displaymultipleitems)? clickindex:forecastlist.length-displaymultipleitems'>
      <block v-for="(item,index) in forecastlist" :key="index">
        <swiper-item @click="clickindex = index" :class="index==clickindex? 'clickindex':''">
            <div>{{item.weekday}}</div>
            <div>{{item.date}}</div>
        </swiper-item>
      </block>
    </swiper>
    <div v-if="forecastlist[clickindex]" class="weatherdetail">
      <div>{{forecastlist[clickindex].condition}}</div>
      <div>{{forecastlist[clickindex].low_temperature}}/{{forecastlist[clickindex].high_temperature}}</div>
      <!-- <div>{{forecastlist[clickindex].high_temperature}}</div> -->
      <div>{{forecastlist[clickindex].wind}}</div>
      <div>{{forecastlist[clickindex].wind_level}}级</div>
    </div>
  </div>
</template>

<script>
import {request} from 'utils';

export default {
  data(){
    return{
      forecastlist:[],
      interval: 5000,
      duration: 1000,
      displaymultipleitems:'4',
      clickindex:null
    }
  },
  onShow:function(){
    this.clickindex  = this.$root.$mp.query.index
  },
  mounted(){
    this.getwwather(this.$root.$mp.query.data.split(','))
  },
  methods: {
    getwwather(localtion){
      request('https://www.toutiao.com/stream/widget/local_weather/data/?city='+localtion[1])
      .then(res=>{
        this.forecastlist = res.data.data.weather.forecast_list.map(o => Object.assign({}, { 'condition': o.condition, 'date': o.date.split('-')[1]+'/'+o.date.split('-')[2], 'high_temperature': o.high_temperature, 'low_temperature': o.low_temperature, 'wind': o.wind_direction, 'wind_level': o.wind_level,'weekday':this.getweeekday(o.date)}))
      })
    },
    getweeekday(date){
      const weekDay = ["星期天", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];
      const myDate = new Date(date);
      return weekDay[myDate.getDay()];
    }
  }
}
</script>

<style scoped>
  @import './index.css';
</style>

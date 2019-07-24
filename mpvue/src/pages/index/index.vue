<template>
  <div class="index">
    <div class="indexbox">
      <div class="localtion">
        <picker class="auth-pick-tip" mode="region" :value="region" @change="regionPick">
          {{region[0]}}，{{region[1]}}，{{region[2]}}
        </picker>
      </div>
      <div class='data_weather'>
        <div class='now_weather'>
          <div class='temperature'>{{dateweather.current_temperature}}℃</div>
          <div class='wind'>{{dateweather.wind_direction}}   {{dateweather.wind_level}}级</div>
        </div>
        <div class='condition'>{{dateweather.current_condition}}</div>
        <div class="quality"><i-tag i-class='quality_level' :color="dateweather.background">{{dateweather.quality_level}}    {{dateweather.aqi}}</i-tag></div>
      </div>
      <!-- 今明两天天气 -->
      <div class='tomorrow_weather'>
        <swiper display-multiple-items="1" :duration="duration" :circular='true'>
          <block v-for="(item,index) in twodateweather" :key="index">
            <swiper-item @click="gotodetail(index*1+1)">
                <div>{{item.date}}</div>
                <div>{{item.condition}}</div>
                <div>{{item.low_temperature}}/{{item.high_temperature}}℃</div>
                <div><i-tag :color="item.color">{{item.quality_level}}</i-tag></div>
            </swiper-item>
          </block>
        </swiper>
      </div>
      <!-- 15天天气 -->
      <div class='halfmonth'>
        <swiper display-multiple-items="4" :duration="duration">
          <block v-for="(item,index) in forecastlist" :key="index">
            <swiper-item @click="gotodetail(index)">
                <div>{{item.weekday}}</div>
                <div>{{item.date}}</div>
                <div>{{item.condition}}</div>
                <div>{{item.low_temperature}}</div>
                <div>{{item.high_temperature}}</div>
                <div>{{item.wind}}</div>
                <div>{{item.wind_level}}级</div>
            </swiper-item>
          </block>
        </swiper>
      </div>
      <!-- 24小时天气 -->
      <div class="hour">
        <swiper display-multiple-items="6" :interval="interval" :duration="duration">
          <block v-for="(item,index) in hourlist" :key="index">
            <swiper-item>
              <div>{{item.condition}}</div>
              <div>{{item.temperature}}</div>
              <div>{{item.hour}}</div>
            </swiper-item>
          </block>
        </swiper>
      </div>
    </div>
  </div>
</template>

<script>
import {request} from 'utils';
import qqMap from '../../../static/js/qqmap-wx-jssdk';
const qqmapsdk = new qqMap({
        key: 'N6JBZ-PVUCV-KJVPE-UYY2R-LZDHZ-DBFKL' //自己的key秘钥 http://lbs.qq.com/console/mykey.html 在这个网址申请
      });
export default {
  data () {
    return {
      region: ['北京市','北京市','东城区'],
      interval: 500,
      duration: 500,
      twodateweather:[],
      // 今天天气信息
      dateweather:{},
      // 15天天气
      forecastlist:[],
      // 24小时天气
      hourlist:[]
    }
  },
  mounted(){
    this.getUserLocation();
  },
  //转发分享
  onShareAppMessage: function(ops) {
    if (ops.from === "button") {
      // 来自页面内转发按钮
      console.log(ops.target);
    }
    return {
      title: "天气",//这里是定义转发的标题
      path: `pages/index/main`,//这里是定义转发的地址
      success: function(res) {
        // 转发成功
        console.log("转发成功:" + JSON.stringify(res));
        var shareTickets = res.shareTickets;
        // if (shareTickets.length == 0) {
        //   return false;
        // }
        // //可以获取群组信息
        // mpvue.getShareInfo({
        //   shareTicket: shareTickets[0],
        //   success: function (res) {
        //     console.log(res)
        //   }
        // })
      },
      fail: function(res) {
        // 转发失败
        console.log("转发失败:" + JSON.stringify(res));
      }
    };
  },
  onPullDownRefresh () {
    this.getUserLocation();
    mpvue.showLoading({
      title: '加载中',
    })
  },
  methods: {
    getUserLocation: function () {
      mpvue.getSetting({
        success: (res) => {
          if (res.authSetting['scope.userLocation'] != undefined && res.authSetting['scope.userLocation'] != true) {
            mpvue.showModal({
              title: '请求授权当前位置',
              content: '需要获取您的地理位置，请确认授权',
              success: (res) => {
                if (res.cancel) {
                  mpvue.showToast({
                    title: '拒绝授权',
                    icon: 'none',
                    duration: 1000
                  })
                } else if (res.confirm) {
                  mpvue.openSetting({
                    success: (dataAu) => {
                      if (dataAu.authSetting["scope.userLocation"] == true) {
                        mpvue.showToast({
                          title: '授权成功',
                          icon: 'success',
                          duration: 1000
                        })
                        //再次授权，调用wx.getLocation的API
                        this.getLocation();
                      } else {
                        mpvue.showToast({
                          title: '授权失败',
                          icon: 'none',
                          duration: 1000
                        })
                      }
                    }
                  })
                }
              }
            })
          } else if (res.authSetting['scope.userLocation'] == undefined) {
            //调用wx.getLocation的API
            this.getLocation();
          }
          else {
            //调用wx.getLocation的API
            this.getLocation();
          }
        }
      })
    },
    // 微信获得经纬度
    getLocation: function () {
      mpvue.getLocation({
        type: 'gcj02',
        success: (res) => {
          var latitude = res.latitude
          var longitude = res.longitude
          var speed = res.speed
          var accuracy = res.accuracy;
          this.getLocal(latitude, longitude)
        },
        fail: function (res) {
          console.log('fail' + JSON.stringify(res))
        },
        altitude:true
      })
    },
    // 获取当前地理位置
    getLocal: function (latitude, longitude) {
      qqmapsdk.reverseGeocoder({
        location: {
          latitude: latitude,
          longitude: longitude
        },
        success: (res) =>{
          this.region = [res.result.address_component.province, res.result.address_component.city, res.result.address_component.district]
          this.getweather(this.region);
        },
        fail:  (res) => {
          console.log(res);
        }
      });
    },
    qualitylevel(level){
      switch (level){
        case '优':
        return 'green';
        break;
        case '良':
          return 'yellow';
          break;
        case '轻度':
          return 'default';
          break;
        case '中度':
          return 'blue';
          break;
        case '重度':
          return 'red';
          break;
      }
    },
    getweather(position){
      request('https://www.toutiao.com/stream/widget/local_weather/data/?city='+position[1])
      .then(res=>{
        this.twodateweather=[
            {
              date: '今天',
              condition: res.data.data.weather.day_condition,
              low_temperature: res.data.data.weather.low_temperature,
              high_temperature: res.data.data.weather.high_temperature,
              quality_level:res.data.data.weather.quality_level,
              color:this.qualitylevel(res.data.data.weather.quality_level)
            },
            {
              date: '明天',
              condition: res.data.data.weather.tomorrow_condition,
              low_temperature: res.data.data.weather.tomorrow_low_temperature,
              high_temperature: res.data.data.weather.tomorrow_high_temperature,
              quality_level: res.data.data.weather.tomorrow_quality_level,
              color:this.qualitylevel(res.data.data.weather.tomorrow_quality_level)
            }
          ],
          this.dateweather={
            current_temperature: res.data.data.weather.current_temperature,
            current_condition: res.data.data.weather.current_condition,
            wind_direction: res.data.data.weather.wind_direction,
            wind_level: res.data.data.weather.wind_level,
            quality_level: res.data.data.weather.quality_level,
            aqi: res.data.data.weather.aqi,
            background:this.qualitylevel(res.data.data.weather.quality_level)
          },
          this.forecastlist = res.data.data.weather.forecast_list.map(o => Object.assign({}, { 'condition': o.condition, 'date': o.date.split('-')[1]+'/'+o.date.split('-')[2], 'high_temperature': o.high_temperature, 'low_temperature': o.low_temperature, 'wind': o.wind_direction, 'wind_level': o.wind_level,'weekday':this.getweeekday(o.date)}))
          this.hourlist = res.data.data.weather.hourly_forecast.map(o=>Object.assign({},{'condition': o.condition,'hour':o.hour>9? o.hour+':00':'0'+o.hour+':00','temperature':o.temperature}))
          setTimeout(()=>{
            mpvue.stopPullDownRefresh();
            mpvue.hideLoading();
          },2000);
          
          //储存选择城市历史记录
          let data = {city:position,weather:res.data.data.weather.day_condition+'  '+res.data.data.weather.low_temperature+'/'+res.data.data.weather.high_temperature}
          let citycook = mpvue.getStorageSync('citycook') || [{}];
          if(![].concat(...(citycook.map(o=>o.city))).includes(position[1])){
            citycook.push(data)
            mpvue.setStorageSync('citycook', citycook.filter(o=>!!Object.values(o).length))
          }
      })
    },
    getweeekday(date){
      const weekDay = ["星期天", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];
      const myDate = new Date(date);
      return weekDay[myDate.getDay()];
    },
    regionPick: function (e) {
      this.region = e.mp.detail.value;
      this.getweather(e.mp.detail.value)
    },
    //去到详情页
    gotodetail(index){
      mpvue.navigateTo({
        url:'../../pages/detail/main?data='+this.region+'&index='+index
      })
    }
  }
}
</script>
<style scoped>
  @import 'index.css';
</style>

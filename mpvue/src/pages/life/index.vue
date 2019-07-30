<template>
  <div class="life">
    <cover-view class="play" @click="tolifedetail">
      {{coverview || '地址、公交查询'}}
    </cover-view>
    <map 
      id="map" 
      :longitude="localtion.longitude" 
      :latitude="localtion.latitude" 
      scale="14" 
      :markers="markers" 
      :polyline='polyline'
      show-location
      include-points
      style="width: 100%; height:100%;">
    </map>
    <cover-view class="bottom" v-if="showbottom">
      <cover-view class="title">{{chooseitem.title}}</cover-view>
      <cover-view class="address">{{chooseitem.address}}</cover-view>
      <button @click="getline(chooseitem.location)">路线</button>
    </cover-view>
  </div>
</template>

<script>
import qqMap from '../../../static/js/qqmap-wx-jssdk';
const qqmapsdk = new qqMap({
        key: 'N6JBZ-PVUCV-KJVPE-UYY2R-LZDHZ-DBFKL' //自己的key秘钥 http://lbs.qq.com/console/mykey.html 在这个网址申请
      });
import vuex from 'store';
let region;
let locationPoint = {};
export default {
  data () {
    return {
      coverview:'',
      markers : [{
        iconPath: "/static/images/location.png",
        id: 0,
        width: 50,
        height: 50
      }],
      localtion:{},
      showbottom:false,
      chooseitem:null,
      polyline: [{
        points:[
          {
            longitude:'',
            latitude: ''
          },
          {
            longitude:'',
            latitude:''
          }
        ],
        color: '#3CB371',
        width: 6
      }],
    }
  },
  onLoad(){
    this.getLocation();
  },
  methods:{
    //获得微信经纬度
    getLocation: function () {
      mpvue.getLocation({
        type: 'gcj02',
        success: (res) => {
          locationPoint = {
            latitude:res.latitude,
            longitude: res.longitude,
          };
          this.getLocal(res.latitude,res.longitude)
        },
        fail: function (res) {
          console.log('fail' + JSON.stringify(res))
        },
        altitude:true
      })
    },
    //获得地址
    getLocal: function (latitude, longitude) {
      qqmapsdk.reverseGeocoder({
        location: {
          latitude: latitude,
          longitude: longitude
        },
        success: (res) =>{
          region = res.result.address_component.city;
          this.markers = [{
            latitude:latitude,
            longitude:longitude,
          }];
          this.localtion = {
            latitude:latitude,
            longitude:longitude
          }
        },
        fail:  (res) => {
          console.log(res);
        }
      });
    },
    tolifedetail(){
      mpvue.navigateTo({
        url:'../../pages/lifedetail/main?city='+region+'&value='+this.coverview,
      })
    },
    getline(location){
      qqmapsdk.direction({
        mode: 'driving',
        from: { latitude: locationPoint.latitude,longitude: locationPoint.longitude},
        to:{ latitude: location.lat,longitude: location.lng},
        success: res => {

          var coors = res.result.routes[0].polyline, pl = [];
          //坐标解压（返回的点串坐标，通过前向差分进行压缩）
          var kr = 1000000;
          for (var i = 2; i < coors.length; i++) {
            coors[i] = Number(coors[i - 2]) + Number(coors[i]) / kr;
          }
          //将解压后的坐标放入点串数组pl中
          for (var i = 0; i < coors.length; i += 2) {
            pl.push({ latitude: coors[i], longitude: coors[i + 1] })
          }

          //设置polyline属性，将路线显示出来,将解压坐标第一个数据作为起点
          this.localtion = {
            latitude:pl[0].latitude,
            longitude:pl[0].longitude
          }
          this.polyline = [{
            points: pl,
            color: '#3CB371',
            width: 6
          }]
        },
        fail: function (error) {
          console.error(error);
        },
        altitude:true
      })
    },
  },
  onShow(){
    //每次显示当前页面时候判断是否有选择的地址要显示
    if(!!Object.values(vuex.state.choosepoint).length){
      this.coverview = vuex.state.choosepoint.title;
      this.localtion = {
        latitude:vuex.state.choosepoint.location.lat,
        longitude:vuex.state.choosepoint.location.lng
      };
      this.markers = [{
        latitude: vuex.state.choosepoint.location.lat,
        longitude: vuex.state.choosepoint.location.lng
      }];
      this.chooseitem = vuex.state.choosepoint;
      this.showbottom = !vuex.state.choosepoint.category.includes('公交线路');
    }
  },
  onHide(){
    //切换到别的tab页面时候，当前页面的信息要清空，地址要默认显示设备所在地址
    Object.assign(this, this.$options.data());
    this.localtion = {
      latitude:locationPoint.latitude,
      longitude:locationPoint.longitude,
    };
    this.markers = [{
      latitude:locationPoint.latitude,
      longitude:locationPoint.longitude,
    }];
    vuex.state.choosepoint = {};
  }
}
</script>

<style scoped>
  @import './index.css';
</style>

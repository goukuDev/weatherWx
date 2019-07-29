<template>
  <div class="life">
    <cover-view class="play" bindtap="play" @click="tolifedetail">
      {{coverview || '地址、公交查询'}}
    </cover-view>
    <map 
      id="map" 
      :longitude="localtion.longitude" 
      :latitude="localtion.latitude" 
      scale="14" 
      :markers="markers" 
      style="width: 100%; height:100%;">
    </map>
  </div>
</template>

<script>
import vuex from 'store';
export default {
  data () {
    return {
      coverview:'',
      markers : [{}],
      localtion:{},
    }
  },
  onLoad(){
    this.markers = [{
      iconPath: "/static/images/location.png",
      id: 0,
      latitude: vuex.state.position[0],
      longitude: vuex.state.position[1],
      width: 50,
      height: 50
    }];
    this.localtion = {
      latitude:vuex.state.position[0],
      longitude:vuex.state.position[1],
      speed:'',
      accuracy:''
    }
  },
  methods:{
    tolifedetail(){
      mpvue.navigateTo({
        url:'../../pages/lifedetail/main?city='+JSON.stringify(vuex.state.location),
      })
    }
  },
  onShow(){
    if(!!Object.values(vuex.state.choosepoint).length){
      this.coverview = vuex.state.choosepoint.title;
      this.localtion = {
        latitude:vuex.state.choosepoint.location.lat,
        longitude:vuex.state.choosepoint.location.lng,
        speed:'',
        accuracy:''
      };
      this.markers = [{
        iconPath: "/static/images/location.png",
        id: 0,
        latitude: vuex.state.choosepoint.location.lat,
        longitude: vuex.state.choosepoint.location.lng,
        width: 50,
        height: 50
      }];
    }
    vuex.state.choosepoint = {};
  },
  onHide(){
    Object.assign(this, this.$options.data());
    this.localtion = {
      latitude:vuex.state.position[0],
      longitude:vuex.state.position[1],
    };
    this.markers = [{
      latitude: vuex.state.position[0],
      longitude: vuex.state.position[1],
    }];
  }
}
</script>

<style scoped>
  @import './index.css';
</style>

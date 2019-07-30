<template>
    <div class="linedetail">
        <cover-view class="play">
            <cover-view v-for="(item,index) in lineType" :key="index" @click="checklinetype(item.type)" :class="{active:chooseLineType==item.type}">{{item.name}}</cover-view>
        </cover-view>
        <map 
        id="map" 
        :longitude="localtion.longitude" 
        :latitude="localtion.latitude" 
        :scale="scale" 
        :markers="markers" 
        :polyline='polyline'
        show-location
        include-points
        style="width: 100%; height:100%;">
        </map>
        <cover-view class="bottom" v-if="showbottom">
            <cover-view style="width:calc(80% - 10px);margin-left:10px;">
                <cover-view class="time">{{showbottom[0].distance>1000? showbottom[0].distance/1000+'千米':showbottom[0].distance+'米'}}   {{showbottom[0].duration}}分钟</cover-view>
                <cover-view class="money" v-if="showbottom[0].taxi_fare">打车约{{showbottom[0].taxi_fare.fare}}元</cover-view>
            </cover-view>
            <cover-view style="width:20%;line-height:80px;text-align:center;font-size:18px;color:blue;">详情</cover-view>
        </cover-view>
    </div>
</template>
<script>
import qqMap from '../../../static/js/qqmap-wx-jssdk';
const qqmapsdk = new qqMap({
        key: 'N6JBZ-PVUCV-KJVPE-UYY2R-LZDHZ-DBFKL'
      });
let from,to;
export default {
    data(){
        return{
            lineType:[
                {
                    type:'driving',
                    name:'驾车'
                },
                {
                    type:'transit',
                    name:'公交'
                },
                {
                    type:'bicycling',
                    name:'骑行'
                },
                {
                    type:'walking',
                    name:'步行'
                }
            ],
            chooseLineType:'driving',
            markers : [{
                iconPath: "/static/images/location.png",
                id: 0,
                width: 50,
                height: 50
            }],
            localtion:{},
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
            showbottom:null,
            scale:'14',
        }
    },
    onLoad(){
        from = JSON.parse(this.$root.$mp.query.localtion);
        to = JSON.parse(this.$root.$mp.query.to);
        this.localtion = {
            latitude:from.latitude,
            longitude:from.longitude
        }
        this.markers = {
            latitude:from.latitude,
            longitude:from.longitude
        }
        //默认显示方式为驾车
        this.getline(from,to,this.chooseLineType)
    },
    methods:{
        getline(from,to,type){
            qqmapsdk.direction({
                mode: type,
                from: { latitude: from.latitude,longitude: from.longitude},
                to:{ latitude: to.lat,longitude: to.lng},
                success: res => {
                //路径规划完成后显示时长、距离等
                this.showbottom = res.result.routes;
                this.scale = 12;
                if(type == 'transit'){
                    var ret = res.result.routes[0];
                    var count = ret.steps.length;
                    var pl = [];
                    var coors = [];
                    //获取各个步骤的polyline
                    for(var i = 0; i < count; i++) {
                        if (ret.steps[i].mode == 'WALKING' && ret.steps[i].polyline) {
                            coors.push(ret.steps[i].polyline);
                        }
                        if (ret.steps[i].mode == 'TRANSIT' && ret.steps[i].lines[0].polyline) {
                            coors.push(ret.steps[i].lines[0].polyline);
                        }
                    }       
                    //坐标解压（返回的点串坐标，通过前向差分进行压缩）
                    var kr = 1000000;
                    for (var i = 0 ; i < coors.length; i++){
                        for (var j = 2; j < coors[i].length; j++) {
                            coors[i][j] = Number(coors[i][j - 2]) + Number(coors[i][j]) / kr;
                        }
                    }
                    //定义新数组，将coors中的数组合并为一个数组
                    var coorsArr = [];
                    for (var i = 0 ; i < coors.length; i ++){
                        coorsArr = coorsArr.concat(coors[i]);
                    }
                    //将解压后的坐标放入点串数组pl中
                    for (var i = 0; i < coorsArr.length; i += 2) {
                        pl.push({ latitude: coorsArr[i], longitude: coorsArr[i + 1] })
                    }
                }else{
                    var ret = res;
                    var coors = ret.result.routes[0].polyline, pl = [];
                    //坐标解压（返回的点串坐标，通过前向差分进行压缩）
                    var kr = 1000000;
                    for (var i = 2; i < coors.length; i++) {
                        coors[i] = Number(coors[i - 2]) + Number(coors[i]) / kr;
                    }
                    //将解压后的坐标放入点串数组pl中
                    for (var i = 0; i < coors.length; i += 2) {
                        pl.push({ latitude: coors[i], longitude: coors[i + 1] })
                    }
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
        checklinetype(type){
            this.chooseLineType=type;
            this.getline(from,to,type)
        },
        onUnload(){
            Object.assign(this, this.$options.data())
        },
    }
}
</script>
<style scoped>
    @import './index.css';
</style>

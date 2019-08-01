<template>
    <div class="searchline">
        <div class="autofocus">
            <input placeholder="地址、公交查询" auto-focus v-model="value" @change="getsuggest(pages)"/>
            <i-icon type="close" size="24" color="#666" v-if="value" @click="value=''"/>
            <!-- <span class="search" @click="search">搜索</span> -->
        </div>
        <div class="pointhistory" v-if="!!checkpoint.length && !value">
            <ul>
                <li v-for="(item,index) in checkpoint" :key="index" class="lists">
                    <div class="pointlist" @click="backfill(item.data)">
                        <!--渲染地址title-->
                        <view>{{item.data.title}}</view>
                        <!--渲染详细地址-->
                        <view>{{item.data.address}}</view>
                    </div>
                    <i-icon type='close' size='24' style="color:rgba(0,0,0,0.4);width:10%;line-height:49px;text-align:center;" @click="deletedone(item._id)"></i-icon>
                </li>
            </ul>
        </div>
        <div class="lifelist" v-if="!!suggestion.length && value">
            <ul>
                <li v-for="item in suggestion" :key="item.id" @click="backfill(item)" class="lists">
                    <!--渲染地址title-->
                    <view :id="item.id">{{item.title}}</view>
                    <!--渲染详细地址-->
                    <view>{{item.address}}</view>
                </li>
                <span @click="getsuggest(pages*1+1)">加载更多</span>
            </ul>
        </div>
    </div>
</template>
<script>
import qqMap from 'static/js/qqmap-wx-jssdk';
const qqmapsdk = new qqMap({
        key: 'N6JBZ-PVUCV-KJVPE-UYY2R-LZDHZ-DBFKL'
      });
import vuex from 'store';
let region;
//初始化数据库
let listdata;
wx.cloud.init();
const db = wx.cloud.database({});
const linelist = db.collection('linelist');
export default {
    data(){
        return{
            suggestion:[],
            value:'',
            pages:1,
            checkpoint:[],
        }
    },
    onLoad(){
        region = this.$root.$mp.query.city;
        this.value = this.$root.$mp.query.value;
    },
    onUnload(){
        Object.assign(this, this.$options.data())
    },
    onShow(){
        this.getlinelist();
    },
    methods:{
        getlinelist(){
            linelist.get({
                success:res=>{
                    this.checkpoint = res.data;
                }
            })
        },
        getsuggest(index){
            if(this.value==''){
                this.suggestion = [];
                return;
            }
            //调用关键词提示接口
            qqmapsdk.getSuggestion({
                keyword:this.value, //用户输入的关键词，可设置固定值,如keyword:'KFC'
                region:region, //设置城市名，限制关键词所示的地域范围，非必填参数
                page_index:index,
                page_size:10,
                success: res => {//搜索成功后的回调
                    this.suggestion.push(...res.data)
                },
                fail: function(error) {
                    console.error(error);
                }
            });
        },
        //数据回填方法
        backfill(item) {
            this.value = item.title;
            //储存搜索历史记录
            listdata = {address:item.address,category:item.category,location:item.location,title:item.title};
            linelist.get({
                success:res=>{
                    if(!res.data.map(o=>o.data).some(o=>o.address==listdata.address && o.title==listdata.title)){
                        linelist.add({
                            data:{
                                data:listdata
                            },
                            success: function(res) {
                                console.log(res._id)
                            }
                        })
                    }
                }
            })
            if(!item.category.includes('公交线路')){
                vuex.state.choosepoint = item;
                mpvue.navigateBack({
                    delta:1,
                })
            }
        },
        // 删除某一个历史数据
        deletedone(id){
            linelist.doc(id).remove({
                success: (res) => {
                    this.getlinelist();
                }
            })
        }
    }
}
</script>
<style scoped>
    @import './index.css';
</style>

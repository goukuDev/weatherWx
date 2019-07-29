<template>
    <div class="lifedetail">
        <div class="autofocus">
            <input placeholder="这是一个可以自动聚焦的input" auto-focus :value="value" @input="getsuggest($event)"/>
            <span class="search">搜索</span>
        </div>
        <div class="lifelist">
            <ul v-if="!!suggestion.length">
                <li v-for="item in suggestion" :key="item.id" @click="backfill(item.id)" class="lists">
                    <!--渲染地址title-->
                    <view :id="item.id">{{item.title}}</view>
                    <!--渲染详细地址-->
                    <view>{{item.address}}</view>
                </li>
            </ul>
        </div>
    </div>
</template>
<script>
import qqMap from '../../../static/js/qqmap-wx-jssdk';
const qqmapsdk = new qqMap({
        key: 'N6JBZ-PVUCV-KJVPE-UYY2R-LZDHZ-DBFKL' //自己的key秘钥 http://lbs.qq.com/console/mykey.html 在这个网址申请
      });
import vuex from 'store';
export default {
    data(){
        return{
            suggestion:[],
            region:[],
            value:''
        }
    },
    onLoad(){
        this.region = JSON.parse(this.$root.$mp.query.city)
    },
    onUnload(){
        Object.assign(this, this.$options.data())
    },
    methods:{
        getsuggest(e){
            if(e.target.value==''){
                this.suggestion = [];
                return;
            }
            //调用关键词提示接口
            qqmapsdk.getSuggestion({
                keyword: e.target.value, //用户输入的关键词，可设置固定值,如keyword:'KFC'
                region:this.region, //设置城市名，限制关键词所示的地域范围，非必填参数
                success: res => {//搜索成功后的回调
                    console.log(res);
                    var sug = [];
                    this.suggestion = res.data
                },
                fail: function(error) {
                    console.error(error);
                }
            });
        },
        //数据回填方法
        backfill(id) {
            vuex.state.choosepoint = this.suggestion.filter(o=>o.id==id)[0];
            this.value = this.suggestion.filter(o=>o.id==id)[0].title;
            mpvue.navigateBack({
                delta:1,
            })
        },
    }
}
</script>
<style scoped>
    @import './index.css';
</style>

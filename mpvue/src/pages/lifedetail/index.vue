<template>
    <div class="lifedetail">
        <div class="autofocus">
            <input placeholder="地址、公交查询" auto-focus v-model="value" @change="getsuggest(pages)"/>
            <!-- <span class="search" @click="search">搜索</span> -->
        </div>
        <div class="lifelist">
            <ul v-if="!!suggestion.length">
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
import qqMap from '../../../static/js/qqmap-wx-jssdk';
const qqmapsdk = new qqMap({
        key: 'N6JBZ-PVUCV-KJVPE-UYY2R-LZDHZ-DBFKL'
      });
import vuex from 'store';
let region;
export default {
    data(){
        return{
            suggestion:[],
            value:'',
            pages:1,
        }
    },
    onLoad(){
        region = this.$root.$mp.query.city;
        this.value = this.$root.$mp.query.value;
    },
    onUnload(){
        Object.assign(this, this.$options.data())
    },
    methods:{
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
            vuex.state.choosepoint = item;
            this.value = item.title;
            if(!vuex.state.choosepoint.category.includes('公交线路')){
                mpvue.navigateBack({
                    delta:1,
                })
            }
        },
    }
}
</script>
<style scoped>
    @import './index.css';
</style>

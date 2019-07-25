<template>
  <div class="kind">
    <i-collapse name="name1" class="kindcollapse" >
        <i-collapse-item title="分类" name="name2" class="kinditem">
            <view slot="content" i-class-content="green-text" class="newslist">
                <div v-for="(item,index) in newslist" :key="index" @click="tonewslist(item.name,item.type)">{{item.name}}</div>
            </view>
        </i-collapse-item>
    </i-collapse>
    <div class="kindbox">
      <div v-for="(item,index) in allTopicList" :key="index" class='kindnews'>
        <title style="font-size:20px;font-weight:600;height:50px;display:block;line-height:50px;">{{item[0].category}}</title>
        <div v-for="(data,key) in item" :key="key" class="newslistdetail" v-if="data.title">
          <div class="left">
            <div class="title">{{data.title}}</div>
            <span style="color:#b4b4b4;font-size:12px;">{{data.source}}</span>
          </div>
          <div class="right" v-if="data.picInfo.length" :style="'background:url('+data.picInfo[0].url+') no-repeat center center;border-radius:3px;background-size:130px 80px;'">
            
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {request} from 'utils';

export default {
  data(){
    return{
      newslist:[
        {
          name:"新闻",
          type:"BBM54PGAwangning"
        },
        {
          name:"娱乐",
          type:"BA10TA81wangning"
        },
        {
          name:"体育",
          type:"BA8E6OEOwangning"
        },
        {
          name:"财经",
          type:"BA8EE5GMwangning"
        },
        {
          name:"军事",
          type:"BAI67OGGwangning"
        },
        {
          name:"科技",
          type:"BA8D4A3Rwangning"
        },
        {
          name:"手机",
          type:"BAI6I0O5wangning"
        },
        {
          name:"汽车",
          type:"BA8DOPCSwangning"
        },
        {
          name:"旅游",
          type:"BEO4GINLwangning"
        },
        {
          name:"教育",
          type:"BA8FF5PRwangning"
        }
      ],
      allTopicList:null,
    }
  },
  onLoad(){
    request('https://3g.163.com/touch/api/pagedata/index_yaowen?callback=allTopicList')
    .then(res=>{
      let data = '{'+res.data.split('allTopicList({')[1].split('})')[0]+'}';
      this.allTopicList = JSON.parse(data).data;
    })
  },
  methods:{
    tonewslist(item,type){
      mpvue.navigateTo({
        url:'../../pages/newslist/main?type='+type
      })
      mpvue.setNavigationBarTitle({
        title:item+'频道'
      })
    }
  }
}
</script>

<style scoped>
  @import './index.css';
</style>

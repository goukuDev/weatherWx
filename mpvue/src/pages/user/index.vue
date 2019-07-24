<template>
  <div class="statistics">
    <div v-if="!citycook.length" class="nothing"> 暂无历史数据</div>
    <div v-else class="history">历史记录</div>
    <div class="citycook" style="height:calc(100% - 40px);overflow-y: auto;-webkit-overflow-scrolling:touch;">
      <div v-for="(item,index) in citycook" :key="index" style="margin-bottom:10px;">
        <i-swipeout  i-class="i-swipeout-demo-item historycity" operateWidth="60">
            <view slot="content">
                {{item.city}}&nbsp;&nbsp;&nbsp;&nbsp;{{item.weather}}℃
            </view>
            <view slot="button" class="i-swipeout-demo-button-group" style="background:#2db7f5;display:flex;height:50px;line-height:50px;text-align:center;">
                <view class="i-swipeout-demo-button" style="width:60px"> <i-icon size="32" type="trash" @click="handlerCloseButton(item,'delete')"></i-icon></view>
                <!-- <view class="i-swipeout-demo-button" style="width:60px"><i-icon size="32" type="undo" @click="handlerCloseButton(item,'back')"></i-icon></view> -->
            </view>
        </i-swipeout>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      citycook:[]
    }
  },
  onShow(){
    this.getstorage()
  },
  methods:{
    getstorage(){
      mpvue.getStorage({
        key:'citycook',
        success:res=>{
          this.citycook = res.data;
        }
      })
    },
    handlerCloseButton(item,data){
      if(data == 'delete'){
        this.citycook = this.citycook.filter(o=>o!=item)
        mpvue.clearStorage();
        mpvue.setStorageSync('citycook',this.citycook.filter(o=>o!=item));
      }
    }
  }
}
</script>

<style scoped>
  @import './index.css';
</style>

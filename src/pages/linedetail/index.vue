<template>
    <div class="linedetail">
        <ul v-if="lineresult.mode">
            <li v-for="(item,index) in lineresult.steps" :key="index">{{item.instruction}}</li>
        </ul>
    </div>
</template>
<script>
export default {
    data(){
        return{
            lineresult:null,
        }
    },
    onLoad(){
        this.lineresult  = JSON.parse(this.$root.$mp.query.data)
        console.log(this.lineresult)
    }
}
</script>
<style scoped>
    @import './index.css';
</style>


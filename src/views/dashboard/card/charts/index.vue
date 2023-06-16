<template>
  <div class="echarts" ref="echarts">

  </div>
</template>

<script>
//1.引入echarts
import * as echarts from 'echarts'
export default {
  name:'',
  mounted() {
    // console.log(echarts);
    //2.初始化echarts实例
    let lineCharts = echarts.init(this.$refs.echarts)
    //3.配置
    lineCharts.setOption({
      xAxis:{
        show:false,
        type:'category'
      },
      yAxis:{
        show:false
      },
      series:{
        type:'line',
        data:[12,32,43,54,23,32,1,13,43,99],
        //拐点
        itemStyle:{
          opacity:0
        },
        lineStyle:{
          color:'purple'
        },
      },
      grid:{
        left:0,
        top:0,
        right:0,
        bottom:0
      }
    })
  },
}
</script>

<style>
  .echarts{
    width: 100%;
    height: 100%;
  }
</style>
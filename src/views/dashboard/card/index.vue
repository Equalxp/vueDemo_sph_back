<template>
  <div>
    <el-row :gutter="10">
      <el-col :span="6">
        <el-card>
          <Detail title="总销售额" count="$152312">
            <template slot="charts">
              <span style="font-size: 10px;">周同比&nbsp;&nbsp;49.92%</span>
              <span style="font-size: 10px;">日同比&nbsp;&nbsp;39.83%</span>
            </template>
            <template slot="footer">
              <span style="font-size:18px">
                日销售额&nbsp;$9231
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <Detail title="访问量" count="2312">
            <template slot="charts">
              <lineCharts></lineCharts>
            </template>
            <template slot="footer">
              <span style="font-size:18px">
                日销售额&nbsp;$9231
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <Detail title="支付笔数" count="912312">
            <template slot="charts">
              <lineCharts></lineCharts>
            </template><template slot="footer">
              <span style="font-size:18px">
                日销售额&nbsp;$9231
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <Detail title="运营活动效果" count="62312">
            <template slot="charts">
              <lineCharts></lineCharts>
            </template><template slot="footer">
              <span style="font-size:18px">
                日销售额&nbsp;$9231
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import lineCharts from './charts'          
import Detail from "./Detail";
export default {
  name: "Card",
  components: {
    Detail,
    lineCharts
  },
};
</script>

<style scoped>

</style>

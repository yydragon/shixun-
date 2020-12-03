<template>
  <div class="mod-demo-echarts">
    <el-alert
      title="提示："
      type="warning"
      :closable="false">
      <div slot-scope="description">
        <p class="el-alert__description">1. 此Demo只提供ECharts官方使用文档，入门部署和体验功能。具体使用请参考：http://echarts.baidu.com/index.html</p>
      </div>
    </el-alert>

    <el-row :gutter="20">
      <el-col :span="24">
        <el-card>
          <div id="J_chartLineBox" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="24">
        <el-card>
          <div id="J_chartBarBox" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <div id="J_chartPieBox" class="chart-box"></div>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card>
          <div id="J_chartScatterBox" class="chart-box"></div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import echarts from 'echarts'
  export default {
    data () {
      return {
        chartLine: null,
        chartBar: null,
        chartPie: null,
		chartScatter: null,
		Allcounts:null
      }
    },
    mounted () {
      this.initChartLine()
      this.initChartBar()
      this.initChartPie()
	  this.initChartScatter()
	  this.getAllCounts()
    },
    activated () {
      // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
      if (this.chartLine) {
        this.chartLine.resize()
      }
      if (this.chartBar) {
        this.chartBar.resize()
      }
      if (this.chartPie) {
        this.chartPie.resize()
      }
      if (this.chartScatter) {
        this.chartScatter.resize()
      }
    },
    methods: {
		getAllCounts(){
			this.$axios.get("/api/echarts/count")
			.then(response=>{
				console.log(response.data.data.data)
				this.Allcounts = response.data.data.data
			})
		},
      // 折线图
      initChartLine () {
       var option = {
			  title: {
				  text: '总览'
			  },
			  tooltip: {},
			  legend: {
				  data:['数量']
			  },
			  xAxis: {
				  data: ["产品","设备","订单","计划","工单"]
			  },
			  yAxis: {},
			  series: [{
				  name: '数量',
				  type: 'bar',
				  data: []
			  }]
		  };
		this.chartLine = echarts.init(document.getElementById('J_chartLineBox'))
		this.$axios.get("/api/echarts/count")
			.then(response=>{
				console.log(response.data.data.data)
				this.chartLine.setOption({
					series: [{
                // 根据名字对应到相应的系列
                name: '数量',
                data: response.data.data.data
            }]
				})

			})
        this.chartLine.setOption(option)
        window.addEventListener('resize', () => {
          this.chartLine.resize()
        })
      },
      // 柱状图
      initChartBar () {
       var option = {
    title: {
        text: '订单状态',
        subtext: '总览',
        left: 'center'
    },
    tooltip: {
        trigger: 'item',
        formatter: '{a} <br/>{b} : {c} ({d}%)'
    },
    legend: {
        orient: 'vertical',
        left: 'left',
        data: ['已拒绝', '未接单', '已拒绝', '生产中', '已完成']
    },
    series: [
        {
            name: '访问来源',
            type: 'pie',
            radius: '55%',
            center: ['50%', '60%'],
            data: [
            ],
            emphasis: {
                itemStyle: {
                    shadowBlur: 10,
                    shadowOffsetX: 0,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                }
            }
        }
    ]
};

        this.$axios.get("/api/echarts/getOrderStatus")
        .then(response=>{
            console.log(response.data.data.data)
            this.chartBar.setOption({
					series: [{
                // 根据名字对应到相应的系列
                name: '访问来源',
                type: 'pie',
                radius: '55%',
                center: ['50%', '60%'],
                data: response.data.data.data 
            }]
				})
        })
        this.chartBar = echarts.init(document.getElementById('J_chartBarBox'))
        this.chartBar.setOption(option)
        window.addEventListener('resize', () => {
          this.chartBar.resize()
        })
      },
      // 饼状图
      initChartPie () {
        var option = {
    title: {
        text: '计划状态',
        subtext: '总览',
        left: 'center'
    },
    tooltip: {
        trigger: 'item',
        formatter: '{a} <br/>{b} : {c} ({d}%)'
    },
    legend: {
        orient: 'vertical',
        left: 'left',
        data: ['未启动', '已启动', '已完成', ]
    },
    series: [
        {
            name: '访问来源',
            type: 'pie',
            radius: '55%',
            center: ['50%', '60%'],
            data: [
            ],
            emphasis: {
                itemStyle: {
                    shadowBlur: 10,
                    shadowOffsetX: 0,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                }
            }
        }
    ]
};

this.$axios.get("/api/echarts/getPlanStauts")
        .then(response=>{
            console.log(response.data.data.data)
            this.chartPie.setOption({
					series: [{
                // 根据名字对应到相应的系列
                name: '访问来源',
                type: 'pie',
                radius: '55%',
                center: ['50%', '60%'],
                data: response.data.data.data
            }]
				})
        })
        this.chartPie = echarts.init(document.getElementById('J_chartPieBox'))
        this.chartPie.setOption(option)
        window.addEventListener('resize', () => {
          this.chartPie.resize()
        })
      },
      // 散点图
      initChartScatter () {
        var option = {
    title: {
        text: '工单状态',
        subtext: '总览',
        left: 'center'
    },
    tooltip: {
        trigger: 'item',
        formatter: '{a} <br/>{b} : {c} ({d}%)'
    },
    legend: {
        orient: 'vertical',
        left: 'left',
        data: ['未开始', '生产中', '已完成',]
    },
    series: [
        {
            name: '访问来源',
            type: 'pie',
            radius: '55%',
            center: ['50%', '60%'],
            data: [
                {value: 2, name: '未开始'},
                {value: 2, name: '生产中'},
                {value: 1, name: '已完成'},
                
            ],
            emphasis: {
                itemStyle: {
                    shadowBlur: 10,
                    shadowOffsetX: 0,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                }
            }
        }
    ]
};
        this.chartScatter = echarts.init(document.getElementById('J_chartScatterBox'))
        this.chartScatter.setOption(option)
        window.addEventListener('resize', () => {
          this.chartScatter.resize()
        })
      }
    }
  }
</script>

<style lang="scss">
  .mod-demo-echarts {
    > .el-alert {
      margin-bottom: 10px;
    }
    > .el-row {
      margin-top: -10px;
      margin-bottom: -10px;
      .el-col {
        padding-top: 10px;
        padding-bottom: 10px;
      }
    }
    .chart-box {
      min-height: 400px;
    }
  }
</style>

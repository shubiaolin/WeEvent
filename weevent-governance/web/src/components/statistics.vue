<template>
  <div class='statistics'>
    <div class='dataContent'>
      <div class='selectOptions'>
        <span class='optionTitle'>选择主题</span>
        <el-select v-model="topic" multiple placeholder="请选择" @visible-change="selectChange" collapse-tags size="small">
          <el-option
            v-for="(item, index) in topicList"
            :key="index"
            :label="item"
            :value="item"
            >
          </el-option>
        </el-select>
        <span class='optionTitle' style='margin-left:20px'>选择时间</span>
        <el-date-picker
          size="small"
          type="daterange"
          value-format="timestamp"
          v-model='selectTime'
          @change="getTime"
          :picker-options="pickerOptions"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期">
        </el-date-picker>
      </div>
      <div class='statisticsCharts'>
        <div class='chart' id='chart'></div>
      </div>
    </div>
  </div>
</template>
<script>
import Highcharts from 'highcharts/highstock'
import { getLastWeek, getTimeList } from '../utils/formatTime'
import API from '../API/resource.js'
export default{
  data () {
    return {
      pickerOptions: {
        disabledDate (time) {
          return time.getTime() > Date.now()
        }
      },
      topicList: [],
      topic: [],
      selectTime: [],
      option: {
        title: {
          text: ''
        },
        yAxis: {
          title: '',
          max: '10',
          lineWidth: 2
        },
        xAxis: {
          categories: []
        },
        tooltip: {
          shared: true,
          crosshairs: true
        },
        series: [],
        legend: {
          align: 'center',
          verticalAlign: 'top',
          y: 20
        },
        credits: {
          enabled: false
        },
        lang: {
          noData: '该时段暂无数据'
        }
      }
    }
  },
  methods: {
    beginDate (e) {
      let vm = this
      let data = {
        'beginDate': vm.selectTime[0],
        'endDate': vm.selectTime[1],
        'topicList': vm.topic,
        'groupId': localStorage.getItem('groupId'),
        'brokerId': localStorage.getItem('brokerId'),
        'userId': localStorage.getItem('userId')
      }
      API.historicalData(data).then(res => {
        if (res.data.status === 200) {
          let resData = res.data.data
          let topic = []
          vm.option.series = []
          if (e) {
            vm.topicList = []
          }
          for (var key in resData) {
            if (e) {
              vm.topicList.push(key)
            }
            topic.push(key)
            let item = {
              'name': key,
              'data': resData[key]
            }
            vm.option.series.push(item)
          }
          let max = 10
          vm.option.series.forEach(x => {
            x.data.forEach(y => {
              max = y > max ? y : max
            })
          })
          vm.option.yAxis.max = max
          vm.topic = [].concat(topic)
          Highcharts.chart('chart', vm.option)
        } else {
          vm.$message({
            type: 'warning',
            message: '数据获取失败'
          })
        }
      })
    },
    getTime (e) {
      let vm = this
      if (!e) {
        vm.getDate()
      } else {
        let timeList = getTimeList(e[0], e[1])
        vm.option.xAxis.categories = [].concat(timeList)
      }
      this.beginDate(true)
    },
    getDate () {
      let data = getLastWeek()
      let start = new Date(data[0]).getTime()
      let end = new Date(data[data.length - 1]).getTime()
      this.selectTime.push(start)
      this.selectTime.push(end)
      this.option.xAxis.categories = [].concat(data)
      this.beginDate(true)
    },
    selectChange (e) {
      if (!e) {
        this.beginDate(false)
      }
    }
  },
  computed: {
    brokerId () {
      return this.$store.state.brokerId
    },
    groupId () {
      return this.$store.state.groupId
    }
  },
  watch: {
    brokerId () {
      this.beginDate(true)
    },
    groupId () {
      this.beginDate(true)
    }
  },
  mounted () {
    this.getDate()
  }
}
</script>

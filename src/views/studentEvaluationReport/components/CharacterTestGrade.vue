<template>
  <div :class="className" :style="{height:height,width:width}"></div>
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import { debounce } from '@/utils'

import { fetchListStudentGrade } from '@/api/StudentGrade'
import { getCurrentUser } from '@/api/user'
export default {
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    },
    characterOneData: {
      type: Number,
      default: null
    },
    characterTwoData: {
      type: Number,
      default: null
    },
    characterThreeData: {
      type: Number,
      default: null
    },
    characterFourData: {
      type: Number,
      default: null
    },
    characterFiveData: {
      type: Number,
      default: null
    }
  },
  data() {
    return {
      chart: null
      // character: this.$storage.get('character')
    }
  },
  mounted() {
    this.getList()
    this.__resizeHanlder = debounce(() => {
      if (this.chart) {
        this.chart.resize()
      }
    }, 100)
    window.addEventListener('resize', this.__resizeHanlder)
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    window.removeEventListener('resize', this.__resizeHanlder)
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    getList() {
      getCurrentUser().then(response => {
        const data = { Sid: response.data.user.sid }
        fetchListStudentGrade(data).then(response => {
          const cstr = JSON.parse(response.data.items[0].scharacter)
          this.characterOneData = cstr[0]
          this.characterTwoData = cstr[1]
          this.characterThreeData = cstr[2]
          this.characterFourData = cstr[3]
          this.characterFiveData = cstr[4]
          this.initChart()
        })
      })
    },
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')
      this.chart.setOption({
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)'
        },
        legend: {
          left: 'center',
          bottom: '10',
          data: ['Industries', 'Technology', 'Forex', 'Gold', 'Forecasts']
        },
        calculable: true,
        series: [
          {
            name: 'WEEKLY WRITE ARTICLES',
            type: 'pie',
            roseType: 'radius',
            radius: [15, 95],
            center: ['50%', '38%'],
            data: [
              { value: this.characterOneData, name: '老虎型' },
              { value: this.characterTwoData, name: '孔雀型' },
              { value: this.characterThreeData, name: '熊猫型' },
              { value: this.characterFourData, name: '猫头鹰型' },
              { value: this.characterFiveData, name: '变色龙型' }
            ],
            animationEasing: 'cubicInOut',
            animationDuration: 2600
          }
        ]
      })
    }
  }
}
</script>

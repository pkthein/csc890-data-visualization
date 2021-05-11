<template>
  <div class="flex flex-center full-height">
    <apexcharts
      v-if="!noData"
      width="100%" height="90%" type="line"
      :options="chartOptions"
      :series="series"
    />
    <div v-else>No data...</div>
  </div>
</template>

<script>
import VueApexCharts from 'vue-apexcharts'

const today = new Date()

export default {
  name: 'Chart',
  components: {
    apexcharts: VueApexCharts
  },
  props: ['state', 'noData', 'iSeries'],
  beforeMount () {
    for (let i = 7; i > 0; i--) {
      let tmp = new Date(today.getTime() - (i * 24 * 60 * 60 * 1000)).toLocaleDateString('en-US')
      tmp = tmp.split('/')
      tmp[tmp.length - 1] = tmp[tmp.length - 1].substring(2)
      tmp = tmp.join('/')
      this.chartOptions.xaxis.categories.push(tmp)
    }

    if (this.noData) {
      this.series = [{
        name: 'series-1',
        data: ['NaN', 'NaN', 'NaN', 'NaN', 'NaN', 'NaN', 'NaN']
      }]
    } else {
      this.series = this.iSeries
    }
  },
  mounted () {
  },
  data: function () {
    return {
      chartOptions: {
        markers: {
          size: 5
        },
        chart: {
          id: 'basic-bar'
        },
        xaxis: {
          categories: []
        },
        dataLabels: {
          // enabled: false,
          style: {
            colors: ['black', '#E91E63', '#9C27B0']
          }
        }
      },
      series: []
    }
  },
  watch: {
    state: function () {
      this.series = this.iSeries
    }
  }
}
</script>

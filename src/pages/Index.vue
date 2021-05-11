<template>
  <q-page class="flex flex-center" style="">
    <div class="full-width q-pa-sm">
      <div class="row q-gutter-x-sm" style="height: 52vh;">
        <q-card bordered flat class="col-8 full">
          <strong class="q-ml-sm text-h6">Map of {{ state ? `${state}, ` : '' }}United States</strong>
          <q-separator />
          <div class="flex flex-center" style="positon: relative; height: 90%; width: 98%; margin: 0 auto;">
            <q-img
              :src="map"
              contain
              height="45vh"
            />
          </div>
        </q-card>
        <q-card bordered flat class="col full">
          <strong class="q-ml-sm text-h6">Filters</strong>
          <q-separator />
          <div class="q-pa-sm q-gutter-y-sm">
            <q-select outlined v-model="state" :options="options" label="State" />
            <q-input outlined v-model="city" label="County" disable />
            <q-btn icon="event" round color="primary">
              <q-popup-proxy @before-show="updateProxy" transition-show="scale" transition-hide="scale">
                <q-date v-model="model" color="orange" range>
                  <div class="row items-center justify-end q-gutter-sm">
                    <q-btn label="Cancel" color="primary" flat v-close-popup />
                    <q-btn label="OK" color="primary" flat @click="save" v-close-popup />
                  </div>
                </q-date>
              </q-popup-proxy>
            </q-btn>
          </div>
        </q-card>
      </div>
      <div class="row q-mt-sm q-gutter-x-sm" style="height: 31vh;">
        <q-card bordered flat class="col full">
          <strong class="q-ml-sm text-h7">Cases/Day (last 7 days)</strong>
          <q-separator />
          <BarChart :state="state" :iSeries="tSeries" />
        </q-card>
        <q-card bordered flat class="col full">
          <strong class="q-ml-sm text-h7">Recovered/Day (last 7 days)</strong>
          <q-separator />
          <BarChart :noData="true" />
        </q-card>
        <q-card bordered flat class="col full">
          <strong class="q-ml-sm text-h7">Deaths/Day (last 7 days)</strong>
          <q-separator />
          <BarChart :state="state" :iSeries="dSeries" :noData="state && dSeries[0].data.length === 0" />
        </q-card>
      </div>
    </div>

    <q-dialog v-model="loading" persistent transition-show="scale" transition-hide="scale">
      <q-card class="bg-primary text-white" style="width: 300px">
        <q-card-section>
          <div class="text-h6 flex flex-center">
            Loading
            <q-spinner-dots
              class="q-ml-xs"
              color="white"
              size="2em"
            /><br>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script>
import BarChart from './BarChart'
import DATA from '../static/data/data.json'

import mockData from '../static/data/mockData'

export default {
  name: 'PageIndex',
  components: {
    BarChart
  },
  beforeMount () {
    this.getUSA()
  },
  mounted () {
  },
  data () {
    return {
      state: null,
      city: null, // not used
      map: require('../../public/USA.png'),
      options: [
        null, 'Alabama', 'Alaska', 'Arizona', 'Arkansas', 'California', 'Colorado', 'Connecticut', 'Delaware', 'Florida', 'Georgia', 'Hawaii', 'Idaho', 'Illinois', 'Indiana', 'Iowa', 'Kansas', 'Kentucky', 'Louisiana', 'Maine', 'Maryland', 'Massachusetts', 'Michigan', 'Minnesota', 'Mississippi', 'Missouri', 'Montana', 'Nebraska', 'Nevada', 'New Hampshire', 'New Jersey', 'New Mexico', 'New York', 'North Carolina', 'North Dakota', 'Ohio', 'Oklahoma', 'Oregon', 'Pennsylvania', 'Rhode Island', 'South Carolina', 'South Dakota', 'Tennessee', 'Texas', 'Utah', 'Vermont', 'Virginia', 'Washington', 'West Virginia', 'Wisconsin', 'Wyoming'
      ],
      mapOptions: {
        null: require('../../public/USA.png'), Alabama: require('../../public/alabama.png'), Alaska: require('../../public/alaska.png'), Arizona: require('../../public/arizona.png'), Arkansas: require('../../public/arkansas.png'), California: require('../../public/california.png'), Nevada: require('../../public/nevada.png')
      },
      dateRange: null,
      model: { from: null, to: null },
      loading: false,
      allData: DATA,
      tSeries: [],
      dSeries: []
    }
  },
  methods: {
    updateProxy () {
      // this.proxyDate = this.date
    },
    save () {
      // this.date = this.proxyDate
    },
    getUSA: function () {
      this.tSeries = [{
        name: 'series-1',
        data: []
      }]
      this.dSeries = [{
        name: 'series-1',
        data: []
      }]

      for (let i = 7; i > 0; i--) {
        this.tSeries[0].data.push(
          this.allData[this.allData.length - i * 2 + 1].totalCases -
          this.allData[this.allData.length - i * 2 - 1].totalCases
        )
        this.dSeries[0].data.push(
          this.allData[this.allData.length - i * 2 + 1].totalDeaths -
          this.allData[this.allData.length - i * 2 - 1].totalDeaths
        )
      }
    },
    getState: function () {
      this.tSeries = [{
        name: 'series-1',
        data: []
      }]
      this.dSeries = [{
        name: 'series-1',
        data: []
      }]

      if (this.state in mockData) {
        this.tSeries[0].data = mockData[this.state].tSeries
        this.dSeries[0].data = mockData[this.state].dSeries
      } else {
        // the data.json does not have the updated data for states
        for (let i = 7; i > 0; i--) {
          const cur = this.allData[this.allData.length - i * 2 + 1].casesByState
          const prev = this.allData[this.allData.length - i * 2 - 1].casesByState

          for (let j = 0; j < cur.length; j++) {
            if (cur[j].name === this.state) {
              this.tSeries[0].data.push(Math.floor((cur[j].casesReported - prev[j].casesReported)))
            }
          }
        }
      }
    }
  },
  watch: {
    state: function () {
      this.map = this.mapOptions[this.state]

      if (this.state) {
        this.getState()
      } else {
        this.getUSA()
      }
    }
  }
}
</script>

<style lang="scss">
.full {
  height: 100%;
  width: 100%;
}
</style>

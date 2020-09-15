<template>
  <div>
    <v-container fluid>
      <v-row>
        <v-col cols="12">
          <div class="grey--text mb-2 text-left">Charts menu</div>
          <v-select
            :items="['Bubble', 'Treemap']"
            v-model="selectMenu"
            label="Solo field"
            solo
          ></v-select>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="6">
          <div class="grey--text mb-2 text-left">Menu 1</div>
          <v-select
            :items="firstMenuItems"
            v-model="selectedItem_1"
            label="Solo field"
            solo
          ></v-select>
        </v-col>
        <v-col cols="6">
          <div class="grey--text mb-2 text-left">Menu 2</div>
          <v-select
            :items="secondMenuItems"
            v-model="selectedItem_2"
            label="Solo field"
            solo
          ></v-select>
        </v-col>
      </v-row>
    </v-container>
    <treemap
      v-if="selectMenu == 'Treemap'"
      :width="800"
      :height="600"
      textColor="white"
      rectColor="blue"
      :data="testData"
    ></treemap>
    <bubble
      v-if="selectMenu == 'Bubble'"
      :width="800"
      :height="400"
      textColor="white"
      rectColor="blue"
      :data="testData"
    ></bubble>
    <!-- <div v-if="forceRender" id="my_dataviz"></div> -->
  </div>
</template>
<script>
import treemap from './treemapD3'
import bubble from './bubbleChart'

export default {
  components: {
    treemap,
    bubble
  },
  data() {
    return {
      render: true,
      selectedItem_1: 'BLLB1 | This tracking BBP.',
      selectedItem_2: 'Q',
      firstMenuData: [
        {
          Name: 'BLLB1',
          Desc: 'This is tracking BBP.',
          URL: 'ws://kundera.ddns.net:9098/ws/svc/run/qry/BLLB1',
          Qkeys: ['name1', 'BBP', 'BBU', 'BBM', 'BBL', 'Q']
        }
      ],
      firstMenuItems: [
        'BLLB1 | This tracking BBP.',
        'ROCDNFAST | This is to track fast down.'
      ],
      secondMenuItems: ['Q', 'BBL', 'BBM'],
      testData: {
        children: [
          {
            name: 'There is no data yet',
            value: 150.65
          }
        ]
      },
      myChartData: [],
      wsUrl: 'ws://kundera.ddns.net:9098/ws/svc/run/qry/BLLB1',
      selectMenu: 'Bubble'
    }
  },
  watch: {
    selectedItem_1: function(val) {
      this.handleMenu_1()
      this.setupWebSocket()
    },
    selectedItem_2: function(val) {
      this.changeData(val)
    },
    myChartData: function(val) {
      this.changeData(val)
    }
  },

  methods: {
    changeData: function(val) {
      let data = this.myChartData
      let flag = false
      data.forEach(obj => {
        if (obj[this.selectedItem_2]) obj.value = obj[this.selectedItem_2]
        else flag = true
        obj.name = obj.Issue
      })
      console.log(data)
      const defaultNoValue = [
        {
          name: 'There is no data yet',
          value: 150.65
        }
      ]
      if (data.length) {
        if (!flag) this.testData.children = data
        else this.testData.children = defaultNoValue
      } else this.testData.children = defaultNoValue

      // console.log(data)

      this.forceRerender()
    },
    handleMenu_1: function() {
      this.myChartData = []
      this.firstMenuData.forEach(d => {
        const key = `${d.Name} | ${d.Desc}`
        if (key == this.selectedItem_1) {
          this.secondMenuItems = d.QKeys
          this.selectedItem_2 = this.secondMenuItems[0]
          this.wsUrl = d.URL
        }
      })
    },
    forceRerender() {
      // Remove my-component from the DOM
      this.render = false

      this.$nextTick(() => {
        // Add the component back in
        this.render = true
      })
    },
    setupWebSocket() {
      console.log('Starting connection to WebSocket Server')
      let connection = new WebSocket(this.wsUrl)
      connection.onmessage = event => {
        this.handleData(event.data)
      }
      connection.onopen = function(event) {
        console.log('Successfully connected to the echo websocket server...')
      }
    },

    handleData: function(data) {
      // console.log(data)
      let x = JSON.parse(data)
      let flag = true
      for (let i = 0; i < this.myChartData.length; i++) {
        if (x.Issue === this.myChartData[i].Issue) {
          this.myChartData[i] = x
          flag = false
        }
      }
      if (flag) {
        this.myChartData.push(x)
      }

      /*************************  Dynamic menu_2 data *****************************/
      // let keys = []
      // for (var key in x) {
      //   // console.log(typeof x[key])
      //   if (typeof x[key] === 'number') keys.push(key)
      // }

      // this.secondMenuItems = keys
    },
    fetchMenuData: async function() {
      const data = await this.$axios.$get(
        'http://kundera.ddns.net:9098/svc/list/shortqry'
      )
      this.firstMenuItems = []
      data.forEach(d => {
        const key = `${d.Name} | ${d.Desc}`
        this.firstMenuItems.push(key)
      })
      this.firstMenuData = data
      this.selectedItem_1 = this.firstMenuItems[0]
    }
  },
  mounted: async function() {
    await this.fetchMenuData()
    this.setupWebSocket()
  }
}
</script>

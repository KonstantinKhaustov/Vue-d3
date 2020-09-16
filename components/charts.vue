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
      v-if="selectMenu == 'Treemap' && render"
      :width="800"
      :height="600"
      textColor="white"
      rectColor="blue"
      :data="testData"
    ></treemap>
    <bubble
      v-if="selectMenu == 'Bubble' && render"
      :width="800"
      :height="400"
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
      showMenu_2: false,
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
      wsConnection: '',
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
      // console.log(this.selectedItem_2)
      data.forEach(obj => {
        (obj.value = obj[this.selectedItem_2]
          ? obj[this.selectedItem_2]
          : 1)
        obj.name = obj.Issue
      })
      if (data.length) this.testData.children = data
      else
        this.testData = {
          children: [
            {
              name: 'There is no data yet',
              value: 150.65
            }
          ]
        }

      // console.log(data)

      this.forceRerender()
    },
    handleMenu_1: function() {
      this.firstMenuData.forEach(d => {
        const key = `${d.Name} | ${d.Desc}`
        if (key == this.selectedItem_1) {
          this.secondMenuItems = d.QKeys
          this.selectedItem_2 = this.secondMenuItems[0]
          this.wsUrl = d.URL
        }
      })
      this.myChartData = []
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
      this.showMenu_2 = false
      if (this.wsConnection) this.wsConnection.close()
      this.wsConnection = new WebSocket(this.wsUrl)

      this.wsConnection.onmessage = event => {
        this.showMenu_2 = true
        this.handleData(event.data)
      }
      this.wsConnection.onopen = function(event) {
        console.log('Successfully connected to the echo websocket server...')
      }
    },

    handleData: function(data) {
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

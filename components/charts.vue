<template>
  <div>
    <v-select :items="items" v-model="selectedItem" label="Solo field" solo></v-select>
    <treemap
      v-if="render"
      :width="600"
      :height="600"
      textColor="white"
      rectColor="blue"
      :data="testData"
    ></treemap>
    <!-- <div v-if="forceRender" id="my_dataviz"></div> -->
  </div>
</template>
<script>
import treemap from './treemapD3'
export default {
  components: {
    treemap
  },
  data() {
    return {
      render: true,
      selectedItem: 'Q',
      items: ['Q', 'BBL', 'BBM'],
      testData: {
        children: [
          {
            name: 'There is no data yet',
            value: 150.65
          }
        ]
      },
      myChartData: []
    }
  },
  watch: {
    selectedItem: function(val) {
      this.changeData(val)
    },
    myChartData: function(val) {
      this.changeData(val)
    }
  },

  methods: {
    changeData: function(val) {
      let data = this.myChartData
      // console.log(this.selectedItem)
      data.forEach(obj => {
        obj.value = obj[this.selectedItem]
        obj.name = obj.Issue
      })
      this.testData.children = data

      // console.log(data)

      this.forceRerender()
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
      let connection = new WebSocket(
        'ws://kundera.ddns.net:9098/ws/svc/run/qry/BLLB1'
      )

      connection.onmessage = event => {
        this.handleData(event.data)
      }

      connection.onopen = function(event) {
        console.log(event)
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

      let keys = []
      for (var key in x) {
        // console.log(typeof x[key])
        if (typeof x[key] === 'number') keys.push(key)
      }

      this.items = keys
    }
  },
  mounted() {
    this.setupWebSocket()
  }
}
</script>
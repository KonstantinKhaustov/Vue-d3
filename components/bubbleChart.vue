<template>
  <div id="my_dataviz"></div>
</template>
<script>
export default {
  props: ['data', 'width', 'height'],

  methods: {
    drawTheChart(
      chartWidth = 600,
      chartHeight = 600,
    ) {
      const root = this.pack(this.data);
      // set the dimensions and margins of the graph
      const margin = { top: 10, right: 10, bottom: 10, left: 10 },
        width = chartWidth - margin.left - margin.right,
        height = chartHeight - margin.top - margin.bottom

      // append the svg object to the body of the page
      const svg = d3
        .select('#my_dataviz')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .attr("font-size", 10)
        .attr("font-family", "sans-serif")
        .attr("text-anchor", "middle");
      const leaf = svg
        .selectAll("g")
        .data(root.leaves())
        .enter()
        .append("g")
        .attr("transform", d => `translate(${d.x + 1},${d.y + 1})`);
      leaf
        .append("circle")
        .attr("id", (d, i) => d.id = "leaf_" + i)
        .attr("r", d=>isNaN(d.r)?0:d.r)
        .attr("fill-opacity", 0.7)
        .attr("fill", d => this.color(d.data.name));
      leaf
        .append("clipPath")
        .attr("id", (d, i) => {
          d.id = "clip_" + i;
          return d.id;
        })
        .append("use")
        .attr("xlink:href", (d, i) => "#leaf_"+i);
      leaf
        .append("text")
        .attr("clip-path", (d, i) => "clip_"+i)
        .selectAll("tspan")
        // .data(d => !isNaN(d.r)?[d.data] : ["no_element"])
        .data(d => !isNaN(d.r)?[d.data.name, d.data.value]:["no_element"])
        .enter()
        .append("tspan")
          .attr("x", 0)
          .attr("y", (d, i, nodes) => `${i - nodes.length / 2 + 0.8}em`)
          .text(d => d);
      leaf
        .append("title")
        .attr("fill", "red")
        .text(
          d =>this.title(d.data)
        );
      /*****************Todo custom tooltip*****************/
      // const tooltip = leaf
      //   .append("g")
      //   .attr("class", "tooltip")
      //   .attr("id", (d, i)=> "tooltip_"+i)
      //   .attr("transform", "translate(20,20)")
      //   .attr("opacity", 0.9)
      // tooltip
      //   .selectAll("rect")
      //   .data(d => [d])
      //     .enter()
      //     .append("rect")
      //     .attr("x", d=>-d.r+7)
      //     .attr("y", d=>-d.r-10)
      //     .attr("rx", 5)
      //     .attr("fill", "white")
      //     .attr("width", d => this.radius(d.data)*2)
      //     .attr("height", 25);
      // tooltip
      //   .append("text")
      //   .attr("fill", "black")
      //   .text(
      //     d =>{
      //       return `${!d.data.name ? "":`${d.data.name} /`}
      //       ${!d.data.value ? "":`${d.data.value}`}`}
      //   );
      //   // leaf.on("mouseover", (event) => {
      //   //   d3.select("> rect > text").attr("opacity", 0);
      //   // })
      //   // leaf.on("mouseout", (event) => {
      //   //   d3.select("> rect > text").attr("opacity", 0.9);
      //   // })
    },
    pack(data) {
      return d3
        .pack()
        .size([this.width - 2, this.height - 2])
        .padding(3)(d3.hierarchy(data).sum(d => {
          let temp = d.value?d.value:0
          return typeof d.value == "number"?Math.abs(temp):d.value;
        }));
    },
    title(data) {
      let str = "";
      for (let key in data) {
        if(!key || key == "name" || key == "value") continue;
          str += `${key} : ${data[key]}\n`
      }
      return str;
    }
  },
  computed: {
    color() {
      return d3.scaleOrdinal()
        .domain(this.data.children.map(d => d.name))
        .range(d3.schemeCategory10);
    }
  },
  mounted() {
    this.drawTheChart(this.width, this.height)
  }
}
</script>
<style lang="scss" scoped>
</style>
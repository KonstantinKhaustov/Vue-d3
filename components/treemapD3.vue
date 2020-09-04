<template>
  <div id="my_dataviz"></div>
</template>
<script>
export default {
  props: ['data', 'width', 'height', 'textColor', 'rectColor'],

  methods: {
    drawTheChart(
      chartWidth = 600,
      chartHeight = 600,
      textColor = 'white',
      rectColor = 'blue'
    ) {
      // set the dimensions and margins of the graph
      var margin = { top: 10, right: 10, bottom: 10, left: 10 },
        width = chartWidth - margin.left - margin.right,
        height = chartHeight - margin.top - margin.bottom

      // append the svg object to the body of the page
      var svg = d3
        .select('#my_dataviz')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')')

      // Give the data to this cluster layout:
      var root = d3.hierarchy(this.data).sum(function(d) {
        return d.value
      }) // Here the size of each leave is given in the 'value' field in input data

      // Then d3.treemap computes the position of each element of the hierarchy
      d3
        .treemap()
        .size([width, height])
        .padding(2)(root)

      svg.selectAll('rect').remove()

      // use this information to add rectangles:
      svg
        .selectAll('rect')
        .data(root.leaves())
        .enter()
        .append('rect')
        .attr('x', function(d) {
          return d.x0
        })
        .attr('y', function(d) {
          return d.y0
        })
        .attr('width', function(d) {
          return d.x1 - d.x0
        })
        .attr('height', function(d) {
          return d.y1 - d.y0
        })
        .style('stroke', 'black')
        .style('fill', rectColor)

      svg.selectAll('text').remove()

      // and to add the text labels
      svg
        .selectAll('text')
        .data(root.leaves())
        .enter()
        .append('text')
        .attr('x', function(d) {
          return d.x0 + 5
        }) // +10 to adjust position (more right)
        .attr('y', function(d) {
          return d.y0 + 20
        }) // +20 to adjust position (lower)
        .text(function(d) {
          return d.data.name
        })
        .attr('font-size', '15px')
        .attr('fill', textColor)
    }
  },
  mounted() {
    this.drawTheChart(this.width, this.height, this.rectColor, this.textColor)
  }
}
</script>
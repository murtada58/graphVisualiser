<script>
    import { onMount } from "svelte";
	import * as d3 from 'd3';
    const euclidieanDistance = (a, b) => Math.sqrt((a*a) + (b*b));
    var data = {
   nodes: [{
     x: 200,
     y: 150
   }, {
     x: 140,
     y: 300
   }, {
     x: 300,
     y: 300
   }, {
     x: 300,
     y: 180
   }],
   links: [{
     source: 0,
     target: 1
   }, {
     source: 1,
     target: 2
   }, {
     source: 2,
     target: 3
   }, {
     source: 3,
     target: 1
   },{
     source: 3,
     target: 0
   }]
 };
 onMount(() => {
    var c10 = d3.scaleOrdinal(d3.schemeCategory10);
 var svg = d3.select("body")
   .append("svg")
   .attr("id", "graphing-area");



 var links = svg.selectAll("link")
   .data(data.links)
   .enter()
   .append("line")
   .attr("class", "link")
   .attr("x1", (l) => data.nodes[l.source].x)
   .attr("y1", (l) => data.nodes[l.source].y)
   .attr("x2", (l) => data.nodes[l.target].x)
   .attr("y2", (l) => data.nodes[l.target].y)
   .attr("stroke", "black");

var lineText = svg.selectAll("link")
   .data(data.links)
   .enter()
   .append("text")
   .attr("x", (l) => (data.nodes[l.source].x + data.nodes[l.target].x) / 2)
   .attr("y", (l) => (data.nodes[l.source].y + data.nodes[l.target].y) / 2)
   .attr('text-anchor', 'middle')
   .attr("class", "lineLength")
   .style('fill', 'red')
   .text((l) => `${euclidieanDistance(data.nodes[l.source].x - data.nodes[l.target].x, data.nodes[l.source].y - data.nodes[l.target].y).toFixed(0)}`);

   var drag = d3.drag()
   .on("drag", function(d, i) {
     d.subject.x = Math.max(0, Math.min(parseFloat(svg.style("width")), d.x));
     d.subject.y = Math.max(0, Math.min(parseFloat(svg.style("height")), d.y));
     d3.select(this).attr("cx", d.subject.x).attr("cy", d.subject.y);
     links.each(function(l) {
         d3.select(this)  
        .attr("x1", (l) => data.nodes[l.source].x)
        .attr("y1", (l) => data.nodes[l.source].y)
        .attr("x2", (l) => data.nodes[l.target].x)
        .attr("y2", (l) => data.nodes[l.target].y)
     });

     lineText.each(function(l) {
        d3.select(this)  
        .attr("x", (l) => (data.nodes[l.source].x + data.nodes[l.target].x) / 2)
        .attr("y", (l) => (data.nodes[l.source].y + data.nodes[l.target].y) / 2)
        .text((l) => `${euclidieanDistance(data.nodes[l.source].x - data.nodes[l.target].x, data.nodes[l.source].y - data.nodes[l.target].y).toFixed(0)}`);
     })
   });

 var nodes = svg.selectAll("node")
   .data(data.nodes)
   .enter()
   .append("circle")
   .attr("class", "node")
   .attr("cx", function(d) {
     return d.x
   })
   .attr("cy", function(d) {
     return d.y
   })
   .attr("r", 15)
   .attr("fill", function(d, i) {
     return c10(i);
   })
   .call(drag); 
 })
  
</script>

<style>
</style>
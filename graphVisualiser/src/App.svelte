<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  const euclidieanDistance = (a, b) => Math.sqrt(a * a + b * b);
  const circleRadius = 20;
  const randomHsl = () => `hsla(200, 100%, 50%, 1)`; // `hsla(${Math.random() * 360}, 100%, 50%, 1)`;
  const modes = {
    move: "move",
    selectNode: "selectNode",
    addNode: "addNode",
    addOrRemoveLink: "addOrRemoveLink",
    removeNode: "removeNode",
  };
  Object.freeze(modes);

  let isgraphDirected = true;
  let isGraphWeighted = true;

  let currentMode = modes.move;
  let selectedNode;

  class Node {
    constructor(value, x, y) {
      this.value = value;
      this.colour = randomHsl();
      this.x = x;
      this.y = y;
      this.links = []; // list of nodes
    }
  }
  class Graph {
    constructor() {
      this.graph;
      this.nodes = [];
    }

    get links() {
      const links = [];
      this.nodes.forEach((nodeFrom) => {
        nodeFrom.links.forEach((nodeTo) => {
          links.push({ source: nodeFrom, target: nodeTo });
        });
      });
      return links;
    }

    update() {
      const self = this;
      this.graph?.remove();
      this.graph = d3.select("body").append("svg").attr("id", "graphing-area");

      document
        .getElementById("graphing-area")
        .addEventListener("click", (event) => {
          if (currentMode !== modes.addNode) {
            return;
          }
          const rect = document
            .getElementById("graphing-area")
            .getBoundingClientRect();
          const x = event.clientX - rect.left; //x position within the element.
          const y = event.clientY - rect.top; //y position within the element.

          self.addNode(0, x, y);
        });

      const markerSize = 10;
      this.graph
        .append("svg:defs")
        .append("svg:marker")
        .attr("id", "arrow")
        .attr("viewBox", "-10 -10 20 20")
        .attr("refX", circleRadius - 3) //so that it comes towards the center.
        .attr("markerWidth", 15)
        .attr("markerHeight", 15)
        .attr("orient", "auto")
        .append("svg:path")
        .attr("d", "M 0,0 m -5,-5 L 5,0 L -5,5 Z");

      const links = this.graph
        .selectAll("link")
        .data(this.links)
        .enter()
        .append("path")
        .attr("class", "link")
        .attr(
          "d",
          (l) =>
            "M" +
            l.source.x +
            "," +
            l.source.y +
            ", " +
            l.target.x +
            "," +
            l.target.y
        )
        .attr("marker-end", isgraphDirected ? "url(#arrow)" : "") //attach the arrow from defs
        .style("stroke", "#000")
        .style("stroke-width", 2);

      const linkText = this.graph
        .selectAll("link")
        .data(this.links)
        .enter()
        .append("text")
        .attr("x", (l) => (l.source.x + l.target.x) / 2)
        .attr("y", (l) => (l.source.y + l.target.y) / 2)
        .attr("text-anchor", "middle")
        .attr("class", "lineLength")
        .style("fill", `rgba(255, 0, 0, ${isGraphWeighted ? "1" : "0"})`)
        .text(
          (l) =>
            `${euclidieanDistance(
              l.source.x - l.target.x,
              l.source.y - l.target.y
            ).toFixed(0)}`
        );

      const drag = d3.drag().on("drag", function (d) {
        if (currentMode === modes.selectNode) {
          selectedNode = d.subject;
          self.update();
        }
        if (currentMode !== modes.move) {
          return;
        }
        d.subject.x = Math.max(
          0,
          Math.min(parseFloat(self.graph.style("width")), d.x)
        );
        d.subject.y = Math.max(
          0,
          Math.min(parseFloat(self.graph.style("height")), d.y)
        );
        d3.select(this).attr("cx", d.subject.x).attr("cy", d.subject.y);
        links.each(function (l) {
          d3.select(this).attr(
            "d",
            (l) =>
              "M" +
              l.source.x +
              "," +
              l.source.y +
              ", " +
              l.target.x +
              "," +
              l.target.y
          );
        });

        linkText.each(function (l) {
          d3.select(this)
            .attr("x", (l) => (l.source.x + l.target.x) / 2)
            .attr("y", (l) => (l.source.y + l.target.y) / 2)
            .text(
              (l) =>
                `${euclidieanDistance(
                  l.source.x - l.target.x,
                  l.source.y - l.target.y
                ).toFixed(0)}`
            );
        });
      });

      this.graph
        .selectAll("node")
        .data(this.nodes)
        .enter()
        .append("circle")
        .attr("class", function (d) {
          if (selectedNode === d) {
            return "node selected-node";
          }
          return "node";
        })
        .attr("cx", function (d) {
          return d.x;
        })
        .attr("cy", function (d) {
          return d.y;
        })
        .attr("r", circleRadius)
        .attr("fill", function (d) {
          return d.colour;
        })
        .on("click", function (d) {
          if (currentMode === modes.selectNode) {
            if (selectedNode === d.target.__data__) {
              selectedNode = undefined;
            } else {
              selectedNode = d.target.__data__;
            }
            self.update();
          } else if (currentMode === modes.removeNode) {
            if (selectedNode === d.target.__data__) {
              selectedNode = undefined;
            }
            self.nodes = self.nodes.filter(
              (node) => node !== d.target.__data__
            );
            self.nodes.forEach(
              (node) =>
                (node.links = node.links.filter(
                  (linkedNode) => linkedNode !== d.target.__data__
                ))
            );
            self.update();
          } else if (currentMode === modes.addOrRemoveLink) {
            if (typeof selectedNode === "undefined") {
              selectedNode = d.target.__data__;
              d3.select(this).attr("class", "node selected-node");
            } else if (selectedNode === d.target.__data__) {
              selectedNode = undefined;
              d3.select(this).attr("class", "node");
            } else {
              if (selectedNode.links.includes(d.target.__data__)) {
                selectedNode.links = selectedNode.links.filter(
                  (linkedNode) => linkedNode !== d.target.__data__
                );
              } else {
                selectedNode.links.push(d.target.__data__);
              }
              self.update();
            }
          }
        })
        .call(drag);
    }

    addNode(value, x, y) {
      selectedNode = new Node(value, x, y);
      this.nodes.push(selectedNode);
      this.update();
    }
  }

  const graph = new Graph();
  onMount(() => {
    graph.update();
  });
</script>

<div id="control-panel">
  <div class="modes">
    <button
      class="mode {currentMode === modes.move ? 'selected-mode' : ''}"
      on:click={() => (currentMode = modes.move)}
    >
      move node
    </button>
    <button
      class="mode {currentMode === modes.selectNode ? 'selected-mode' : ''}"
      on:click={() => (currentMode = modes.selectNode)}
    >
      select node
    </button>
    <button
      class="mode {currentMode === modes.addNode ? 'selected-mode' : ''}"
      on:click={() => (currentMode = modes.addNode)}
    >
      add node
    </button>
    <button
      class="mode {currentMode === modes.addOrRemoveLink
        ? 'selected-mode'
        : ''}"
      on:click={() => (currentMode = modes.addOrRemoveLink)}
    >
      add/remove link
    </button>
    <button
      class="mode {currentMode === modes.removeNode ? 'selected-mode' : ''}"
      on:click={() => (currentMode = modes.removeNode)}
    >
      remove node
    </button>

    <button
      class="mode {isGraphWeighted ? 'selected-mode' : ''}"
      on:click={() => {
        isGraphWeighted = !isGraphWeighted;
        graph.update();
      }}
    >
      {isGraphWeighted ? "weighted" : "unweighted"}
    </button>

    <button
      class="mode {isgraphDirected ? 'selected-mode' : ''}"
      on:click={() => {
        isgraphDirected = !isgraphDirected;
        graph.update();
      }}
    >
      {isgraphDirected ? "directed" : "undirected"}
    </button>
  </div>
</div>

<div id="node-data">
  <h2>{selectedNode?.colour}</h2>
</div>

<style>
  .selected-mode {
    background-color: blue;
    color: red;
  }
</style>

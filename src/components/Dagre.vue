<template>
  <div>
  </div>
</template>

<script>
import * as d3 from "d3";
import dagreD3 from "dagre-d3";
import tippy from "tippy.js";
import "tippy.js/dist/tippy.css";

export default {
  props: {
    tree: String
  },
  data:
      function () {
        return {
          svg: {}
        }
      },
  computed: {
    nodes() {
      var treeJson = JSON.parse(this.tree)
      var result = [];
      Object.keys(treeJson.nodes).forEach(function (key) {
        result.push({id: key, label: key});
      });
      return result
    },
    edges() {
      var treeJson = JSON.parse(this.tree)
      console.log(treeJson);
      var map = new Object();
      Object.keys(treeJson.nodes).forEach(function (key) {
        var node = treeJson.nodes[key];
        if (node.outcomeTrue) {
          if (map[key + "_" + node.outcomeTrue.nextNode] === undefined) {
            map[key + "_" + node.outcomeTrue.nextNode] = {from: key, to: node.outcomeTrue.nextNode, label: "True"};
          } else {
            map[key + "_" + node.outcomeTrue.nextNode].label = map[key + "_" + node.outcomeTrue.nextNode].label + "\n" + "True";
          }
        }
        if (node.outcomeFalse) {
          if (map[key + "_" + node.outcomeFalse.nextNode] === undefined) {
            map[key + "_" + node.outcomeFalse.nextNode] = {
              from: key,
              to: node.outcomeFalse.nextNode,
              label: "False"
            };
          } else {
            map[key + "_" + node.outcomeFalse.nextNode].label = map[key + "_" + node.outcomeFalse.nextNode].label + "\n" + "False";
          }
        }
        if (node.outcomeMissing) {
          if (map[key + "_" + node.outcomeMissing.nextNode] === undefined) {
            map[key + "_" + node.outcomeMissing.nextNode] = {
              from: key,
              to: node.outcomeMissing.nextNode,
              label: "Missing"
            };
          } else {
            map[key + "_" + node.outcomeMissing.nextNode].label = map[key + "_" + node.outcomeMissing.nextNode].label + "\n" + "Missing";
          }
        }
        if (node.outcomeDefault) {
          if (map[key + "_" + node.outcomeDefault.nextNode] === undefined) {
            map[key + "_" + node.outcomeDefault.nextNode] = {
              from: key,
              to: node.outcomeDefault.nextNode,
              label: "Default"
            };
          } else {
            map[key + "_" + node.outcomeDefault.nextNode].label = map[key + "_" + node.outcomeDefault.nextNode].label + "\n" + "Default";
          }
        }
        if (node.outcomeMap) {
          Object.keys(node.outcomeMap).forEach(
              function (cat) {
                if (map[key + "_" + node.outcomeMap[cat].nextNode] === undefined) {
                  map[key + "_" + node.outcomeMap[cat].nextNode] = {
                    from: key,
                    to: node.outcomeMap[cat].nextNode,
                    label: cat
                  };
                } else {
                  map[key + "_" + node.outcomeMap[cat].nextNode].label = map[key + "_" + node.outcomeMap[cat].nextNode].label + "\n" + cat;
                }
              });
        }
      });
      var result = [];
      for (const edge in map) {
        result.push(map[edge]);
      }
      return result;
    }
  },
  mounted() {
    this.render(this.nodes, this.edges);
  },
  watch: {
    nodes: function () {
      this.refresh();
    },
    edges: function () {
      this.refresh();
    }
  },
  methods: {
    render(nodes, edges) {
      var g = new dagreD3.graphlib.Graph().setGraph({});
      nodes.forEach(function (node) {
        g.setNode(node.id, {label: node.label});
        if (node.id.indexOf("error") === -1) {
          g.node(node.id).style = "fill: #7f7";
        } else {
          g.node(node.id).style = "fill: #f77";
        }
      });
      edges.forEach(function (edge) {
        g.setEdge(edge.from, edge.to, {label: edge.label});
      });

      g.nodes().forEach(function (v) {
        var node = g.node(v);
        node.rx = node.ry = 5;
      });

      this.svg = d3
          .select(this.$el)
          .append("svg")
          .attr("width", document.body.clientWidth - 350)
          .attr("height", 250);
      var inner = this.svg.append("g");
      // Set up zoom support
      var zoom = d3.zoom().on("zoom", function () {
        inner.attr("transform", d3.event.transform);
      });
      this.svg.call(zoom);

      var render = new dagreD3.render();
      render(inner, g);

      inner
          .selectAll("g.node")
          .attr("title", function () {
            return "hello world";
          })
          .each(function (v) {
            tippy(this, {
              content: v,
              interactive: true,
              allowHTML: true,
              appendTo: document.body
            });
          });

      var initialScale = 0.65;
      this.svg.call(
          zoom.transform,
          d3.zoomIdentity
              .translate(
                  (this.svg.attr("width") - g.graph().width * initialScale) / 2,
                  20
              )
              .scale(initialScale)
      );

      this.svg.attr("height", g.graph().height * initialScale + 40);
    },
    refresh() {
      this.svg.remove();
      this.render(this.nodes, this.edges)
    }
  }
};
</script>


<style>
h1,
svg {
  text-align: center;
}

svg {
  display: block;
  margin: auto;
  border: 1px solid #ccc;
}

.node rect {
  stroke: #333;
  fill: #fff;
}

.edgePath path {
  stroke: #333;
  fill: #333;
  stroke-width: 1.5px;
}
</style>

<template>
  <div :id="id"
       class="tree-container">
    <svg class="d3-tree">
      <g class="container"></g>
    </svg>
  </div>
</template>
<script>
import dataset from './dataset2.js'
export default {
  name: 'Scale',
  data () {
    return {
      id: '',
      zoom: null,
      index: 0,
      duration: 750,
      root: null,
      nodes: [],
      links: [],
      dTreeData: null,
      transform: null,
      margin: { top: 20, right: 90, bottom: 30, left: 90 }
    }
  },
  computed: {
    treemap () {
      return d3.tree().size([this.height, this.width])
    }
  },
  created () {
    this.id = this.uuid()
  },
  mounted () {
    // 创建svg画布
    this.width = document.getElementById(this.id).clientWidth
    this.height = document.getElementById(this.id).clientHeight

    const svg = d3.select(this.$el).select('svg.d3-tree')
      .attr('width', this.width)
      .attr('height', this.height)

    // const transform = d3.zoomIdentity.translate(this.margin.left, this.margin.top).scale(1)
    const transform = d3.zoomIdentity.translate(this.width / 2, this.margin.top).scale(1)
    const container = svg.select('g.container')

    // 初始化缩放行为
    this.zoom = d3.zoom()
      .scaleExtent([1 / 2, 8])
      .on('zoom', this.zoomed)

    container.transition().duration(750).call(this.zoom.transform, transform)

    svg.call(this.zoom)

    this.root = this.getRoot()

    this.update(this.root)
  },
  methods: {
    uuid () {
      function s4 () {
        return Math.floor((1 + Math.random()) * 0x10000)
          .toString(16)
          .substring(1)
      }
      return (
        s4() + s4() + '-' + s4() + '-' + s4() + '-' + s4() + '-' + s4() + s4() + s4()
      )
    },
    /**
     * @description 获取构造根节点
     */
    getRoot () {
      let root = d3.hierarchy(dataset, d => d.children)
      root.x0 = this.height / 2
      root.y0 = 0
      return root
    },
    /**
     * @description 点击节点，展开or收缩
     */
    clickNode (d) {
      if (!d._children && !d.children)
        return
      if (d.children) {
        this.$set(d, '_children', d.children)
        d.children = null
      } else {
        this.$set(d, 'children', d._children)
        d._children = null
      }
      this.$nextTick(
        () => {
          this.update(d)
        }
      )
    },
    diagonal (s, d) {
      return `
        M ${s.y} ${s.x}
        C ${(s.y + d.y) / 2} ${s.x},
        ${(s.y + d.y) / 2} ${d.x},
        ${d.y} ${d.x}
      `
    },
    /**
     * @description 获取构造的node数据和link数据
     */
    getNodesAndLinks () {
      // 树状图根据根节点生成新的x、y坐标
      this.dTreeData = this.treemap(this.root)
      this.nodes = this.dTreeData.descendants()
      this.links = this.dTreeData.descendants().slice(1)
    },
    /** 
     * @description 数据与Dom进行绑定
     */
    update (source) {
      this.getNodesAndLinks()
      console.log(this.nodes)
      /*  this.nodes.forEach((d, index) => {
        d.y = d.depth * 180
      }) */
      this.nodes.forEach((d, index) => {
        d.y = d.depth * -180
      })
      function parseChildren(nodes) {
        for(var i=0; i< nodes.length; i++) {
          if (this.children) {
            
          } else {

          }
        }
      }
      parseChildren(this.nodes)
      const svg = d3.select(this.$el).select('svg.d3-tree')
      const container = svg.select('g.container')
      let node = container.selectAll('g.node')
        .data(this.nodes, d => {
          return d.id || (d.id = ++this.index)
        })
      // Enter any new sources at the parent's previous position.
      let nodeEnter = node.enter().append('g')
        .attr('class', 'node')
        .on('click', this.clickNode)
        .attr('transform', d => {
          return 'translate(' + source.y0 + ',' + source.x0 + ')'
        })

      nodeEnter.append("circle")
        .attr("r", 5)
        .style("fill", function (d) { return d.children || d._children ? "lightsteelblue" : "#fff" })

      nodeEnter.append("text")
        .attr("x", function (d) { return d.children || d._children ? -15 : 15; })
        .attr("dy", ".35em")
        .attr("text-anchor", function (d) { return d.children || d._children ? "end" : "start"; })
        .text(function (d) { return d.data.name })
        .style("fill-opacity", 1e-6);
      // 变换节点到一个新的位置.
      let nodeUpdate = nodeEnter.merge(node)
        .transition()
        .duration(this.duration)
        .attr("transform", function (d) { return "translate(" + d.y + "," + d.x + ")"; });

      nodeUpdate.select("circle")
        .attr("r", 5)
        .attr("fill", "white")
        .attr("stroke", "blue")
        .attr("stroke-width", 1)
        .style("fill", function (d) { return d.children || d._children ? "lightsteelblue" : "#fff"; });

      nodeUpdate.select("text")
        .style("fill-opacity", 1);

      // Transition exiting nodes to the parent's new position.
      let nodeExit = node.exit()
        .transition()
        .duration(this.duration)
        .attr("transform", function (d) { return "translate(" + source.y + "," + source.x + ")"; })
        .remove();

      nodeExit.select("circle")
        .attr("r", 1e-6);

      nodeExit.select("text")
        .style("fill-opacity", 1e-6);

      // Update the links…
      let link = container.selectAll('path.link')
        .data(this.links, d => { return d.id })

      // Enter any new links at the parent's previous position.
      let linkEnter = link.enter().insert("path", "g")
        .attr("class", "link")
        .attr("d", d => {
          let o = {
            x: source.x0,
            y: source.y0
          };
          return this.diagonal(o, o)
        })
        .attr("fill", 'none')
        .attr("stroke-width", 1)
        .attr('stroke', '#ccc')
      // Transition links to their new position.
      let linkUpdate = linkEnter.merge(link)
      linkUpdate.transition()
        .duration(this.duration)
        .attr('d', d => { return this.diagonal(d, d.parent) })

      // Transition exiting nodes to the parent's new position.
      link.exit().transition()
        .duration(this.duration)
        .attr("d", d => {
          let o = { x: source.x, y: source.y };
          return this.diagonal(o, o)
        })
        .remove();

      // 保存旧的位置，以便过渡
      this.nodes.forEach(d => {
        d.x0 = d.x
        d.y0 = d.y
      })
    },
    /** 
     * @description 控制画布放大与缩小
     */
    zoomed () {
      d3.select(this.$el).select('g.container').attr('transform', d3.event.transform)
    }
  }
}
</script>
<style lang='scss' scoped>
.tree-container {
  width: 100%;
  height: calc(100vh - 84px);
}
.d3-tree {
  width: 100%;
  height: 100%;
  .node {
    cursor: pointer;
  }
  .node circle {
    fill: #fff;
    stroke: steelblue;
    stroke-width: 1.5px;
  }
  .node text {
    font: 18px sans-serif;
  }
  .link {
    fill: none;
    stroke: #ccc;
    stroke-width: 1.5px;
  }
}
</style>

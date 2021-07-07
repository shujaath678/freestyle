<template>
  <div id="app" class="container-fluid">
    <div class="col-md-9 panel panel-default">
      <tree ref="tree"
            v-model="currentData"
            :nodeTextDisplay="nodeTextDisplay"
            :identifier="getId"
            :nodeTextMargin="nodeTextMargin"
            :zoomable="zoomable"
            :data="Graph.tree"
            :leafTextMargin="leafTextMargin"
            :node-text="nodeText"
            :margin-x="Marginx"
            :margin-y="Marginy"
            :radius="radius"
            :type="type"
            :layout-type="layoutType"
            :linkLayout="linkLayout"
            :duration="duration"
            :minZoom="minZoom"
            :maxZoom="maxZoom"
            contextMenuPlacement="bottom-start"
            class="tree"
            @clickedText="onClick"
            @expand="onExpand"
            @retract="onRetract"
            @clickedNode="onClickNode">

        <template #popUp="{data,node}">
          <div class="btn-group-vertical">
            <button type="button" class="btn btn-primary" @click="addFor(data)" data-toggle="tooltip"
                    title="Add children">
              <i class="fa fa-plus" aria-hidden="true"></i>
            </button>
            <button type="button" class="btn btn-danger" @click="remove(data, node)" data-toggle="tooltip"
                    title="Remove node">
              <i class="fa fa-trash" aria-hidden="true"></i>
            </button>
          </div>
        </template>
      </tree>
    </div>

  </div>
</template>

<script>
import {tree} from 'vued3tree'
import {getGremlin} from 'vued3tree'
// import noBehavior from 'vued3tree'
let currentId = 500

const removeElement = (arr, element) => {
  const index = arr.indexOf(element)
  if (index === -1) {
    return
  }
  arr.splice(index, 1)
}

const data = {
  "Graph": {
    "tree": {
      "children": [
        {
          'id': 1,
          'text': "NEW",
          "children": [
            {
              'id': 2,
              'text': 'Honda',
              "children": [
                {
                  "children":
                      [
                        {
                          "children": [],
                          "id": 53,
                          "text": "DiscogsEntityType"
                        },
                        {
                          "children": [],
                          "id": 50,
                          "text": "DiscogsImageFormatType"
                        }, {
                        "children": [],
                        "id": 52, "text": "DiscogsPaginable"
                      }, {
                        "children": [],
                        "id": 54,
                        "text": "DiscogsSearch"
                      }, {"children": [], "id": 51, "text": "DiscogsSortInformation"}], "id": 49, "text": "Query"
                }]
            },
            {
              'id': 2,
              'text': 'Hyundai',
              "children": [
                {
                  "children":
                      [{"children": [], "id": 53, "text": "DiscogsEntityType"}, {
                        "children": [],
                        "id": 50,
                        "text": "DiscogsImageFormatType"
                      }, {"children": [], "id": 52, "text": "DiscogsPaginable"}, {
                        "children": [],
                        "id": 54,
                        "text": "DiscogsSearch"
                      }, {"children": [], "id": 51, "text": "DiscogsSortInformation"}], "id": 49, "text": "Query"
                }]
            }
          ],
        },
        {
          'id': 1,
          'text': "USED",
          "children": [
            {
              'id': 2,
              'text': 'Honda',
              "children": [
                {
                  "children":
                      [{"children": [], "id": 53, "text": "DiscogsEntityType"}, {
                        "children": [],
                        "id": 50,
                        "text": "DiscogsImageFormatType"
                      }, {"children": [], "id": 52, "text": "DiscogsPaginable"}, {
                        "children": [],
                        "id": 54,
                        "text": "DiscogsSearch"
                      }, {"children": [], "id": 51, "text": "DiscogsSortInformation"}], "id": 49, "text": "Query"
                }]
            }
          ],
        }]
    },
    "links": [],
    "text": "DiscogsClient"
  }
};

Object.assign(data, {
  type: 'tree',
  layoutType: 'horizontal',
  duration: 750,
  Marginx: 30,
  Marginy: 30,
  radius: 10,
  leafTextMargin: 15,
  nodeTextMargin: 15,
  nodeText: 'text',
  currentData: null,
  zoomable: true,
  isLoading: false,
  isUnderGremlinsAttack: false,
  nodeTextDisplay: 'all',
  linkLayout: 'bezier',
  minZoom: 0.8,
  maxZoom: 9,
  events: []
})

export default {
  name: 'app',
  data() {
    return data
  },
  components: {
    tree,
    // noBehavior
  },
  methods: {
    async do(action) {
      if (this.currentData) {
        this.isLoading = true
        await this.$refs['tree'][action](this.currentData)
        this.isLoading = false
      }
    },
    getId(node) {
      return node.id
    },
    expandAll() {
      this.do('expandAll')
    },
    collapseAll() {
      this.do('collapseAll')
    },
    showOnly() {
      this.do('showOnly')
    },
    show() {
      this.do('show')
    },
    onClick(evt) {
      this.onEvent('clickedText', evt)
    },
    onClickNode(evt) {
      this.onEvent('clickedNode', evt)
    },
    onExpand(evt) {
      this.onEvent('onExpand', evt)
    },
    onRetract(evt) {
      this.onEvent('onRetract', evt)
    },
    onEvent(eventName, data) {
      this.events.push({eventName, data: data.data})
    },
    addFor(data) {
      const newData = {
        id: currentId++,
        children: [],
        text: Math.random().toString(36).substring(7)
      }
      data.children.push(newData)
    },
    remove(data, node) {
      const parent = node.parent.data
      removeElement(parent.children, data)
    },
    resetZoom() {
      if (!this.$refs['tree']) {
        return
      }
      this.isLoading = true
      this.$refs['tree'].resetZoom().then(() => {
        this.isLoading = false
      })
    },
    gremlins() {
      if (this.isUnderGremlinsAttack) {
        this.horde.stop()
        return
      }

      const updateType = (type) => {
        switch (type) {
          case 'vertical':
            return 'circular'

          case 'circular':
            return 'horizontal'

          case 'horizontal':
            return 'vertical'
        }
      }

      const updateNodeTextDisplay = (display) => {
        switch (display) {
          case 'all':
            return 'leaves'

          case 'leaves':
            return 'extremities'

          case 'extremities':
            return 'all'
        }
      }

      this.duration = 20
      const changeLayout = () => {
        this.type = (this.type === 'tree') ? 'cluster' : 'tree'
      }
      const changeNode = () => {
        this.linkLayout = (this.linkLayout === 'bezier') ? 'orthogonal' : 'bezier'
      }
      const changeType = () => {
        this.layoutType = updateType(this.layoutType)
      }
      const changeNodeTextDisplay = () => {
        this.nodeTextDisplay = updateNodeTextDisplay(this.nodeTextDisplay)
      }
      const resetZoom = this.resetZoom.bind(this)
      const [treeDiv] = this.$el.getElementsByClassName('tree')
      const [gremlinsButton] = this.$el.getElementsByClassName('btn-danger')
      var horde = getGremlin(gremlinsButton, treeDiv, {
        changeType,
        changeLayout,
        changeNode,
        changeNodeTextDisplay,
        resetZoom
      })
      horde.after(() => {
        this.isUnderGremlinsAttack = false
      })
      horde.unleash()
      this.horde = horde
      this.isUnderGremlinsAttack = true
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #0e7cea;
  margin-top: 20px;
}

.tree {
  height: 800px;
}

.graph-root {
  height: 800px;
  width: 100%;
}

.feedback {
  height: 50px;
  line-height: 50px;
  vertical-align: middle;
}

.log {
  height: 200px;
  overflow-x: auto;
  overflow-y: auto;
  overflow: auto;
  text-align: left;
}

text {
  font-family: 'Roboto', 'Helvetica Neue', 'Helvetica', 'PingFang SC',
  'Hiragino Sans GB', 'Microsoft YaHei',Arial, sans-serif !important;
  font-size: 12px;
  fill: #08a2f5;
}

circle {
  fill: none !important;
  stroke: #08a2f5;
  stroke-width: 3;
}

path {
  stroke: #08a2f5 !important;
}
</style>
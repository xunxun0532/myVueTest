<template>
  <div class="flow-node" :draggable="nodeInfo.draggable" :style="nodePosition" @dragstart="nodeDragStart(option.id)" @drag="nodeDragging(option.id)" @dragend="nodeDragEnd(option.id)" @mouseenter="mouseEnterNode(option.id)">
    <icon :name="option.type" :size="80" :ref="option.id" style="cursor:move" />
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex';

export default {
  name: 'node',
  data() {
    return {
      arrowVisible: false,
      nodeInfo: {
        draggable: true
      }
    };
  },
  props: {
    option: {
      type: Object
    }
  },
  computed: {
    ...mapState('flow', ['nodeData', 'lineData']),
    nodePosition() {
      return {
        left: `${this.option.left}px`,
        top: `${this.option.top}px`
      };
    }
  },
  methods: {
    ...mapMutations('flow', [
      'UPDATE_NODE',
      'UPDATE_HOVER_NODE',
      'UPDATE_LINE'
    ]),
    nodeDragStart(id) {
      let img = this.$el;
      let dataTransfer = event.dataTransfer;
      dataTransfer.dropEffect = 'move';
      dataTransfer.setData('Text', `update:${this.option.id}`);
      dataTransfer.setDragImage(img, 40, 40);
      this.$emit('nodeDragStart', id);
    },
    nodeDragging(id) {
      // this.$emit('nodeDragging', { id, event });
      const x = event.clientX;
      const y = event.clientY;
      const { width, height } = this.nodeData[id];
      const nodeOffsetX = x- 208;
      const nodeOffsetY = y- 68 -height;
      this.updateLine(Number(nodeOffsetX), Number(nodeOffsetY), id);
    },
    updateLine(nodeOffsetX, nodeOffsetY, id) {
      let data = {};
      for (var key in this.lineData) {
        const {startId,endId} = this.lineData[key]

        if (startId === id) {
          data = {
            ...data,
            [key]: {
              ...this.lineData[key],
              startPosition: {
                x: nodeOffsetX,
                y: nodeOffsetY
              }
            }
          };
        } else if (endId === id) {
          data = {
            ...data,
            [key]: {
              ...this.lineData[key],
              endPosition: {
                x: nodeOffsetX,
                y: nodeOffsetY
              }
            }
          };
        }
      }
      this.UPDATE_LINE(data);
    },
    nodeDragEnd(id) {
      this.$emit('nodeDragEnd', { id, event });
    },
    mouseEnterNode(id) {
      this.UPDATE_HOVER_NODE({
        id
      });
    },
    getNodeSize() {
      let id = this.option.id;
      let { width, height, x, y } = this.$refs[id].$el.childNodes[0].getBBox();
      width = +width.toFixed(0);
      height = +(height - 3).toFixed(0);
      x = +x.toFixed(0);
      y = +y.toFixed(0);

      this.UPDATE_NODE({
        [id]: {
          ...this.nodeData[id],
          width,
          height,
          x,
          y
        }
      });
    },
    init() {
      let id = this.option.id;
      if (typeof this.nodeData[id].width === 'undefined') {
        this.getNodeSize();
      }
    }
  },
  mounted() {
    this.init();
  }
};
</script>

<style lang="scss">
.flow-node {
  position: absolute;
  z-index: 9999;
  &.flow-node:hover a {
    border-right: 1px solid #000;
  }
}
</style>
<template>
  <TransitionGroup ref="dragArea" name="list" tag="div" class="drag-sort">
    <div
      ref="cardRef"
      class="card"
      v-for="(item, index) in list"
      :key="item.name"
      :style="{ width: item.width, height: item.height }"
      @dragenter="handleDragEnter(index, $event)"
      @dragover="handleDragOver"
      @mousedown="activeCard($event)"
    >
      <!-- 头部拖拽条 -->
      <div
        class="drag-position-bar"
        @dragstart="handleDragStart(index, $event)"
        draggable="true"
      ></div>
      <!-- 内容区 -->
      <div class="card-content">
        业务逻辑
        {{ item.name }}
      </div>
      <!-- 拖拽右下角的icon -->
      <div
        class="drag-scale-icon"
        @mousedown="handleMouseDown($event, item)"
      ></div>
    </div>
  </TransitionGroup>
</template>
<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      list: [
        { name: "a", width: "800px", height: "200px" },
        { name: "b" },
        { name: "c" },
        { name: "d" },
        { name: "e" },
      ],
      // 鼠标按下时的坐标
      oldX: 0,
      oldY: 0,
      // 拖拽是旧的元素信息
      activeDragEl: {},
      oldWidth: 0,
      oldHeight: 0,
      oldIndex: 0,
      lock: false,

      // 容器的宽高
      containerWidth: 0,
      containerHeight: 0,
    };
  },
  computed: {},
  created() {},
  mounted() {
    console.log(this.$slots.default);
  },
  methods: {
    activeCard(e) {
      this.$refs.cardRef.forEach((item) => {
        item.classList.remove("active");
      });
      e.currentTarget.classList.add("active");
    },
    handleDragOver(e) {
      e.preventDefault();
    },
    handleDragEnter(index, e) {
      e.preventDefault();
      if (index === this.oldIndex) return;
      if (this.lock) return;
      const source = this.list[this.oldIndex];
      this.list.splice(this.oldIndex, 1);
      this.list.splice(index, 0, source);
      this.oldIndex = index;
      this.lock = true;
      setTimeout(() => {
        this.lock = false;
      }, 200);
    },
    handleDragStart(index, e) {
      e.dataTransfer.setDragImage(e.target.parentNode, e.offsetX, 0);
      this.oldIndex = index;
    },

    handleMouseDown(e, item) {
      // 记录鼠标按下时的坐标 & dom的宽高
      this.oldX = e.clientX;
      this.oldY = e.clientY;
      this.activeDragEl = e.target.parentNode;
      const { width, height } = this.activeDragEl.getBoundingClientRect();
      this.oldWidth = width;
      this.oldHeight = height;

      // 记录容器的宽高
      const { width: containerWidth, height: containerHeight } =
        this.$refs.dragArea.$el.getBoundingClientRect();
      this.containerWidth = containerWidth;
      this.containerHeight = containerHeight;

      const abortSignal = new AbortController();
      window.addEventListener(
        "mousemove",
        (e) => this.handldeMouseMove(e, item),
        {
          signal: abortSignal.signal,
        }
      );
      window.addEventListener(
        "mouseup",
        () => {
          console.log(item.name);
          //记录拖动后的位置信息
          item.width = this.activeDragEl.style.width;
          item.height = this.activeDragEl.style.height;
          abortSignal.abort();
        },
        {
          signal: abortSignal.signal,
        }
      );
    },
    handldeMouseMove(e, item) {
      const { clientX, clientY } = e;
      const { oldX, oldY } = this;
      const diffX = clientX - oldX;
      const diffY = clientY - oldY;
      // console.log("move", item);

      // px记录
      // this.activeDragEl.style.height = this.oldHeight + diffY + "px";
      // this.activeDragEl.style.width = this.oldWidth + diffX + "px";
      // 百分比记录
      this.activeDragEl.style.height =
        ((this.oldHeight + diffY) / this.containerHeight) * 100 + "%";
      this.activeDragEl.style.width =
        ((this.oldWidth + diffX) / this.containerWidth) * 100 + "%";
    },
  },
};
</script>
<style lang='scss' scoped>
.drag-sort {
  height: 100%;
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;

  .card {
    height: 200px;
    width: 300px;
    margin: 0 10px 10px 0;
    background: white;
    display: flex;
    flex-direction: column;
    position: relative;
    border: 2px solid rgb(222, 224, 227);
    border-radius: 10px;
    &:hover {
      border: 2px solid #c2d4ff;
      .drag-scale-icon {
        display: block;
      }
    }
    .card-content {
      flex: 1;
    }
    .drag-position-bar {
      height: 20px;
      border-radius: 8px 10px 0 0;
      height: 20px;
      width: 100%;
      cursor: grab;
      background: black;
    }
    .drag-scale-icon {
      display: none;
      background: rgb(20, 86, 240);
      border: 2px solid rgb(255, 255, 255);
      border-radius: 50%;
      height: 12px;
      width: 12px;
      position: absolute;
      right: 0;
      bottom: 0;
      border-radius: 50%;
      transform: translate(50%, 50%);
      z-index: 10;
      cursor: nwse-resize;
    }
  }
  .active {
    border: 2px solid rgb(20, 86, 240) !important;

    .drag-scale-icon {
      display: block;
    }
  }
}
.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.2s ease;
}
</style>
<template>
  <TransitionGroup ref="dragArea" name="list" tag="div" class="drag-sort">
    <div
      ref="cardRef"
      class="card"
      v-for="(item, index) in list"
      :key="item.id"
      :style="{ width: item.width, height: item.height }"
      @dragenter="handleDragEnter(index, $event)"
      @dragover="handleDragOver"
      @mousedown="activeCard($event)"
    >
      <!-- 头部拖拽条 -->
      <div
        class="drag-position-bar"
        @dragstart="handleDragStart(index, $event)"
        @dragend="handleDragEnd(item)"
        draggable="true"
      >
        <div class="drag-bar-left">
          <div class="drag-icon">:::</div>
          {{ item.headerName }}
        </div>
        <div class="drag-bar-right">
          <el-dropdown trigger="click" @command="(c) => handleCommand(c, item)">
            <span class="drag-icon"> ... </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                v-for="item in commandList"
                :key="item.command"
                :icon="item.iconClass"
                :command="item.command"
                >{{ item.commandName }}</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
        </div>
      </div>
      <!-- 内容区 -->
      <slot :name="`slot-${index}`"></slot>
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
  props: {
    value: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      list: this.value,
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
      // 命令列表
      commandList: [
        {
          command: "edit",
          commandName: "编辑",
          iconClass: "",
        },
        {
          command: "copy",
          commandName: "复制",
        },
        {
          command: "delete",
          commandName: "删除",
        },
      ],
    };
  },
  watch: {
    value(val) {
      this.$emit("updateData", {
        type: "listChange",
        item: val[val.length - 1],
      });
    },
  },
  methods: {
    handleCommand(command, item) {
      this.$emit("operate", {
        command,
        item,
      });
    },
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
      }, 150);
    },
    handleDragStart(index, e) {
      e.dataTransfer.setDragImage(e.target.parentNode, e.offsetX, 0);
      this.oldIndex = index;
    },
    handleDragEnd(item) {
      //拖拽结束，通知外界更新数据
      this.$emit("updateData", {
        type: "drag",
        item,
      });
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
          //记录拖动后的位置信息
          item.width = this.activeDragEl.style.width;
          item.height = this.activeDragEl.style.height;
          // 缩放结束，通知外界更新数据
          this.$emit("updateData", {
            type: "scale",
            item,
          });
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
      // 高度采用px,宽度采用百分比记录

      // px记录
      this.activeDragEl.style.height = this.oldHeight + diffY + "px";
      // this.activeDragEl.style.width = this.oldWidth + diffX + "px";
      // 百分比记录
      // this.activeDragEl.style.height =
      //   ((this.oldHeight + diffY) / this.containerHeight) * 100 + "%";
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
    .drag-position-bar {
      height: 20px;
      border-radius: 8px 10px 0 0;
      height: 20px;
      background: white;
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding-left: 15px;
      padding-right: 15px;
      height: 32px;
      font-size: 16px;
      cursor: grab;
      .drag-bar-left {
        display: flex;
        .drag-icon {
          transform: rotate(90deg);
          margin-right: 10px;
        }
      }
      .drag-bar-right {
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        .drag-icon {
          padding-left: 5px;
          padding-right: 5px;
        }
        &:hover {
          background-color: rgba(31, 35, 41, 0.05);
        }
      }
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
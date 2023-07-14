<template>
  <div class="contain">
    <div id="app">
      <dragSort
        v-model="list"
        @operate="handleOperate"
        @updateData="handleUpdateData"
      >
        <template v-for="(item, index) in list" v-slot:[`slot-${index}`]>
          <!-- Echarts图表 -->
          <div class="item" :key="item.id">{{ item.id }}</div>
        </template>
      </dragSort>
      <div @click="handleAdd">添加图表</div>
    </div>
  </div>
</template>

<script>
import { nanoid } from "nanoid";
import dragSort from "./components/dragSort.vue";
export default {
  components: {
    dragSort,
  },
  name: "App",
  data() {
    return {
      list: [
        {
          id: nanoid(),
          headerName: "图表1",
          width: "calc(50% - 15px)",
          height: "200px",
        },
        {
          id: nanoid(),
          headerName: "图表2",
          width: "calc(50% - 15px)",
          height: "200px",
        },
        {
          id: nanoid(),
          headerName: "图表3",
          width: "calc(50% - 15px)",
          height: "200px",
        },
        {
          id: nanoid(),
          headerName: "图表4",
          width: "calc(50% - 15px)",
          height: "200px",
        },
        {
          id: nanoid(),
          headerName: "图表5",
          width: "calc(50% - 15px)",
          height: "200px",
        },
      ],
    };
  },
  created() {
    let storeList = JSON.parse(localStorage.getItem("listData"));
    if (storeList) {
      this.list = storeList;
    }
  },
  methods: {
    handleAdd() {
      this.list.push({
        id: nanoid(),
        headerName: `图表${this.list.length + 1}`,
        width: "calc(50% - 15px)", //半屏
        height: "200px",
      });
    },
    // 用户点击了:复制/编辑/删除
    handleOperate(data) {
      switch (data.command) {
        case "edit":
          console.log("编辑");
          break;
        case "copy":
          console.log("复制");
          break;
        case "delete":
          this.list.splice(
            this.list.findIndex((item) => item.id === data.item.id),
            1
          );
          console.log("删除");
          break;
      }
    },
    // 更新数据: 缩放/拖拽/list发生变动
    handleUpdateData(data) {
      switch (data.type) {
        case "scale":
          console.log("缩放结束", data);
          break;
        case "drag":
          console.log("拖拽结束", data);
          break;
        case "listChange":
          console.log("图表数量变动", data);
          break;
      }
      localStorage.setItem("listData", JSON.stringify(this.list));
    },
  },
};
</script>

<style lang="scss" scoped>
.contain {
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    // height: 800px;
    padding-top: 10px;
  }
  .item {
    height: 100%;
    width: 100%;
  }
}
</style>

<template>
  <ul>
    <li
      v-for="(item,index) in list"
      :key="index"
      class="getItem"
      :data-shape="item.shape"
      :data-type="item.type"
      :data-size="item.size"
      draggable
      @dragstart="handleDragstart"
      @dragend="handleDragEnd($event,item)"
    >
      <span class="pannel-type-icon" :style="{background:'url('+item.image+')'}"></span>
      {{item.name}}
    </li>
  </ul>
</template>

<script>
import eventBus from "@/utils/eventBus";
import okSvg from "@/assets/icons/ok.svg";
// import bgImg from "@/assets/bg.jpg";
export default {
  
  data() {
    return {
      page: null,
      command: null,
      offsetX: 0,
      offsetY: 0,
      router: 1,
      network: 1,
      list: [
        {
          name: "路由器",
          label: "路由器",
          error: false,
          delete: false,
          size: "130*34",
          bgcolor:'#A0CFFF',
          type: "circle",
          x: 0,
          y: 0,
         shape: 'customNode',
          color: "#1890ff",
         
          stateImage: !this.error?okSvg:'',
          inPoints: [[0, 0.5]],
          outPoints: [[1, 0.5]],
          routingTable: [{to:'-',next:'-',distance:'-'}]
        },

        {
          name: "网络",
          label: "网络",
          error: false,
          delete:false,
          size: "170*34",
          bgcolor:"#fff",
          type: "node",
          x: 0,
          y: 0,
          shape: "customNode",
          color: "#1890ff",
         
          stateImage: okSvg,
          inPoints: [[0, 0.5]],
          outPoints: [[1, 0.5]]
        },
      ]
    };
  },
  
  created() {
    this.bindEvent();
  },
  methods: {
    
    handleDragstart(e) {
      this.offsetX = e.offsetX;
      this.offsetY = e.offsetY;
    },
    handleDragEnd(e, item) {
      let data = {};
      Object.assign(data, item);
      data.offsetX = this.offsetX;
      data.offsetY = this.offsetY;
      if (this.page) {
        const graph = this.page.graph;
        // const size = e.target.dataset.size.split("*");
        const xy = graph.getPointByClient(e.x, e.y);
        data.x = xy.x;
        data.y = xy.y;
        data.size = item.size.split("*");
        if(item.label == '路由器')
          data.label = item.label + this.router++
        else
          data.label = item.label + this.network++
        data.type = "node";
        console.log(data)
        this.command.executeCommand("add", [data]);
      }
    },
    bindEvent() {
      eventBus.$on("afterAddPage", page => {
        this.page = page;
        this.command = page.command;
      });
    }
  }
};
</script>

<style scoped>
.itempannel {
  height: 100%;
  position: absolute;
  left: 0px;
  z-index: 2;
  background: #f7f9fb;
  width: 200px;
  padding-top: 8px;
  border-right: 1px solid #e6e9ed;
}
.itempannel ul {
  padding: 0px;
  padding-left: 16px;
}
.itempannel li {
  color: rgba(0, 0, 0, 0.65);
  border-radius: 4px;
  width: 160px;
  height: 28px;
  line-height: 26px;
  padding-left: 8px;
  border: 1px solid rgba(0, 0, 0, 0);
  list-style-type: none;
}
.itempannel li:hover {
  background: white;
  border: 1px solid #ced4d9;
  cursor: move;
}

.itempannel .pannel-type-icon {
  width: 16px;
  height: 16px;
  display: inline-block;
  vertical-align: middle;
  margin-right: 8px;
}
</style>
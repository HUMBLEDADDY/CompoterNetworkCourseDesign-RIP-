<template>
  <div class="detailpannel">
    <div>
      <div v-if="status=='node-selected'" class="pannel" id="node_detailpannel">
        <div class="pannel-title">模型详情</div>
        <div class="block-container">
          <el-row :gutter="10">
            <el-col :span="8">名称</el-col>
            <el-col :span="16">
              <el-input v-model="node.label" @change="handleChangeName" />
            </el-col>
            <el-col :span="8">故障</el-col>
            <el-col :span="16">
              <el-checkbox v-model="node.error" @change="updateError">故障啦</el-checkbox>
            </el-col>
          </el-row>
        </div>
      </div>
      <div v-if="status==='canvas-selected'" class="pannel" id="canvas_detailpannel">
        <div class="pannel-title">画布</div>
        <div class="block-container">
          <el-checkbox v-model="showGrid" @change="changeGridState">网格对齐</el-checkbox><br/>
          
        </div>
      </div>
      <!-- <div v-if="status==='group-selected'" class="pannel" id="node_detailpannel">
        <div class="pannel-title">群组详情</div>
        <div class="block-container">
          <div class="p">
            名称：
            <el-input v-model="name" />
          </div>
          <div class="p">
            任意属性：
            <el-input v-model="color" />
          </div>
        </div>
      </div>
      -->
    </div>
  </div>
</template>

<script>
import eventBus from "@/utils/eventBus";
import Grid from "@antv/g6/build/grid";
export default {
  data() {
    return {
      status: "canvas-selected",
      showGrid: false,
      page: {},
      graph: {},
      item: {},
      node: {},
      grid: null,
      routingTables: [],
      theRipGraph: {},
      theNetwork: [],
      theRouter: [],
      edges: [],
    };
  },
  created() {
    this.init();
    this.bindEvent();
  },
  methods: {
    init() {},
    bindEvent() {
      let self = this;
      eventBus.$on("afterAddPage", page => {
        self.page = page;
        self.graph = self.page.graph;
        eventBus.$on("nodeselectchange", item => {
          if (item.select === true && item.target.getType() === "node") {
            self.status = "node-selected";
            self.item = item.target;
            self.node = item.target.getModel();
          } else {
            self.status = "canvas-selected";
            self.item = null;
            self.node = null;
          }
        });
      });
    },
    handleChangeName(e) {
      const model = {
        label: e
      };

      this.graph.update(this.item, model);
    },
    changeGridState(value) {
      if (value) {
        this.grid = new Grid();
        this.graph.addPlugin(this.grid);
      } else {
        this.graph.removePlugin(this.grid);
      }
    },
    start(){
      console.log('Start To Simulate The Rip！！！ :)')
      var myVar = setInterval(()=>{
        var theRipGraph = this.graph.save()
        if(JSON.stringify(theRipGraph) != JSON.stringify(this.theRipGraph)){
          this.updategraph(theRipGraph)
        }
        var routingTables = this.routingTables
        for(let i=0;i<routingTables.length;i++){
          for(let j=0;j<routingTables.length;j++){
            if(i!=j){
              
              for(let k=0;k<routingTables[i].routingTable.length;k++){
                if(routingTables[i].routingTable[k].distance != 16 && routingTables[j].routingTable[k].distance != 16 ){
                  
                  for(let l =0 ; l < routingTables[i].routingTable.length;l++){
                    if(routingTables[i].routingTable[l].distance+1 < routingTables[j].routingTable[l].distance){
                      console.log('meimaob')
                      routingTables[j].routingTable[l].distance = routingTables[i].routingTable[l].distance+1
                      routingTables[j].routingTable[l].next = routingTables[i].label
                    }
                  }
                }
              }
            }
          }
        }
        this.routingTables = routingTables
        console.log(this.routingTables)
      }, 1000);
    },
    updategraph(theRipGraph){
      var theNetwork = []
      var theRouter = []
      var nodes = theRipGraph.nodes
      var edges = theRipGraph.edges
      // console.log(theRipGraph)
      //导入节点
      for(let i=0;i<nodes.length;i++){//遍历出路由器和网络
        if(nodes[i].name==='网络')
          theNetwork.push(nodes[i])
        else if(!nodes[i].error)
          theRouter.push(nodes[i])
      }
      // console.log(theNetwork)
      var routingTables = new Array();//路由表数组(伪二维数组)
      for(let i = 0; i < theRouter.length; i++ ){
        var routerId = theRouter[i].id//放入路由器的id,便于遍历边
        var routerLable = theRouter[i].label//放入路由器的id,便于遍历边
        let routingTable = new Array();//路由表(单表)
        for(let j = 0; j < theNetwork.length; j++ ){
          let target = {targetName:theNetwork[j].label,next:'-',distance:16,id:theNetwork[j].id }//one row of ratingtable
          routingTable.push(target)
        }
        // console.log(routingTable)
        routingTables.push({routerId,routerLable,routingTable});
      }
      // console.log(routingTables)

      //导入边
      var edgesTable = []
      for(let i=0;i<edges.length;i++){//遍历
        var edge = {from:edges[i].source,to:edges[i].target}
        edgesTable.push(edge)
      }
      for(let  i=0;i<edgesTable.length;i++){
        for(let j=0;j<routingTables.length;j++){
          if(routingTables[j].routerId == edgesTable[i].from || routingTables[j].routerId == edgesTable[i].to){
            for(let k=0;k<theNetwork.length;k++){
              if(routingTables[j].routingTable[k].id == edgesTable[i].from || routingTables[j].routingTable[k].id == edgesTable[i].to){
                routingTables[j].routingTable[k].distance = 1
                // console.log(routingTables[j].routerId)
                // console.log(routingTables[j].routingTable[k])
              }
            }
          }
        }
      }
      console.log(routingTables)
      this.routingTables = routingTables
      this.theRipGraph = JSON.parse(JSON.stringify(theRipGraph))
    },
    updateError(e){
      console.log(e)
      if(e){
        const model = {
          stateImage: "",
          bgcolor:'#F56C6C',
          color:'#F56C6C'
        }
        
        this.graph.update(this.item, model);
        console.log(this.item)
      }
      else{
         const model = {
          stateImage: "/static/img/ok.463ab0e4.svg",
          bgcolor:'#A0CFFF',
          color:'#1890FF'
        }
        this.graph.update(this.item, model);
      }

    }
  }
};
</script>

<style scoped>
.detailpannel {
  height: 44vh;
  position: absolute;
  right: 0px;
  z-index: 2;
  background: #f7f9fb;
  width: 200px;
  border-left: 1px solid #e6e9ed;
}
.detailpannel .block-container {
  padding: 16px 8px;
}
.block-container{
  display: flex;
  flex-direction: column;
  align-items: center;
}
.block-container .el-col {
  height: 28px;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.pannel-title {
  height: 32px;
  border-top: 1px solid #dce3e8;
  border-bottom: 1px solid #dce3e8;
  background: #ebeef2;
  color: #000;
  line-height: 28px;
  padding-left: 12px;
}
</style>

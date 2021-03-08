<template>
    <div class="tables">
        <el-button class="start" type="primary" @click='start'>开始模拟</el-button>
				<div  class="routetables" >
				<div class="routetable" v-for="(index) in routingTables" :key="index.routerId">
            <div class="name">{{index.routerLable}}</div>
            <el-table
                :data="index.routingTable"
                style="width: 300px">
                <el-table-column
                    prop="targetName"
                    label="目标网络"
                    width="100">
                </el-table-column>
                <el-table-column
                    prop="next"
                    label="下一跳"
                    width="100">
                </el-table-column>
                <el-table-column
                    prop="distance"
                    label="距离">
                </el-table-column>
            </el-table>
        </div>
				</div>
        
    </div>
</template>

<script>
import eventBus from "@/utils/eventBus";
import Grid from "@antv/g6/build/grid";
export default {
  data() {
    return {
      graph: {},
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
							let k=0;
							var check =setInterval(()=>{
								if(k<routingTables[i].routingTable.length){
									if(routingTables[i].routingTable[k].distance != 16 && routingTables[j].routingTable[k].distance != 16 ){
										for(let l =0 ; l < routingTables[i].routingTable.length;l++){
											if(routingTables[i].routingTable[l].distance+1 < routingTables[j].routingTable[l].distance){
												console.log('meimaob')
												routingTables[j].routingTable[l].distance = routingTables[i].routingTable[l].distance+1
												routingTables[j].routingTable[l].next = routingTables[i].routerLable
												this.routingTables = routingTables
											}
										}
									}
									k++
								}
							},1000)
            } 
          }
        }
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
  }
};
</script>
<style>
.name{
	text-align: center;
	background-color: #A0CFFF;
	color: #fff;
}
.tables{
	padding: 20px;
    margin-bottom: 50px;
		height: fit-content;
}
    .start{
        margin: auto;
        display: block;
        position: absolute;
    }
		.routetables{
				margin-bottom: 100px;
				display: flex;
				flex-direction: row;
				flex-wrap: wrap;
				height: fit-content;
    } 
    .routetable{
        position: relative;
        top: 30px;
				display: block;
				width: 300px;
        margin: 20px;
        background-color: #f5f5f5;
        border-radius: 10px;
        overflow: hidden;
        transition: 1s;
        box-shadow: 3px 5px 5px rgba(129, 129, 129, 0.5);
    }    
    .routetable:hover{
        transition: 1s;

        box-shadow: 6px 10px 10px rgba(194, 194, 194, 0.24);
    }    
</style>
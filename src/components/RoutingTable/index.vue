<template>
    <div class="tables">
        <el-button class="start" type="primary" @click='start'>开始模拟</el-button>
				<div  class="routetables" >
				<div class="routetable" v-for="(index) in theShowRoutingTables" :key="index.routerId">
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
      theShowRoutingTables: [],
      theRipGraph: {},
      theNetwork: [],
      theRouter: [],
      edgesTable: [],
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
      var haschange = false
      var myVar = setInterval(()=>{
        var theRipGraph = this.graph.save()
        if(JSON.stringify(theRipGraph) != JSON.stringify(this.theRipGraph)){
          this.updategraph(theRipGraph)
          haschange = true
        }
        var routingTables = this.routingTables
        var theRouter = this.theRouter
        var theNetwork = this.theNetwork
        for(let i=0;i<routingTables.length;i++){
          for(let j=0;j<routingTables.length;j++){
            if(i!=j){
                let k=0;
							var check =setInterval(()=>{
								if(k<routingTables[i].routingTable.length){
									if(routingTables[i].routingTable[k].distance == 1 ){
                    routingTables[i].routingTable[k].next = '-'
                  
                  if(routingTables[j].routingTable[k].distance == 1 ){
                    
										for(let l =0 ; l < routingTables[i].routingTable.length;l++){
											if(routingTables[i].routingTable[l].distance+1 < routingTables[j].routingTable[l].distance){
												// console.log('meimaob')
												routingTables[j].routingTable[l].distance = routingTables[i].routingTable[l].distance+1
												routingTables[j].routingTable[l].next = routingTables[i].routerLable
                        // console.log(routingTables[j].routerLable+'走'+routingTables[i].routerLable+'到'+routingTables[j].routingTable[l].targetName+'距离'+routingTables[j].routingTable[l].distance)
                        // this.routingTables = JSON.parse(JSON.stringify(routingTables))

											}
										}
									}
                  }
  
                      for(let m=0;m<routingTables.length;m++){
                      for(let n=0;n<routingTables[m].routingTable.length;n++){

                          if(!this.checkpath(routingTables[m].routerId,routingTables[m].routingTable[n].id)){
                            routingTables[m].routingTable[n].distance = 16
                            routingTables[m].routingTable[n].next = '-'
                            // console.log('awdwd')
                          }
                        
                      }
                    }
                    // console.log('hvg')

                    console.log(routingTables)


                        var theShowRoutingTables = []
                        for(let n=0;n<routingTables.length;n++){
                          let theShowRoutingTable = []
                          for(let q=0;q<routingTables[n].routingTable.length;q++){
                            for(let w=0;w<theNetwork.length;w++){
                              if(theNetwork[w].id == routingTables[n].routingTable[q].id&&(!(theNetwork[w].delete))){
                                theShowRoutingTable.push(routingTables[n].routingTable[q])
                                break;
                              }
                            }

                          }
                            routingTables[n].routingTable = JSON.parse(JSON.stringify(theShowRoutingTable))
                          if(!(routingTables[n].isDelete)){
                            theShowRoutingTables.push(routingTables[n])
                          }
                        }
                        this.theShowRoutingTables = JSON.parse(JSON.stringify(theShowRoutingTables))
									k++
								}
							},500)
            } 
          }
        }
        // console.log(this.routingTables)
      }, 500);
    },
    updategraph(theRipGraph){
      var theNetwork = []
      var theRouter = []
      var nodes = theRipGraph.nodes
      var edges = theRipGraph.edges
      var routingTables = this.routingTables;
      var hasadd  = false
      // console.log(theRipGraph)
      //导入节点
      for(let i=0;i<nodes.length;i++){//遍历出路由器和网络
        if(nodes[i].name==='网络')
          theNetwork.push(nodes[i])
        else
          theRouter.push(nodes[i])
      }

      // console.log(theNetwork)
      // console.log(JSON.stringify(routingTables))
      if(JSON.stringify(routingTables) == '[]'){
        // console.log('ok')
        hasadd =true
        for(let i = 0; i < theRouter.length; i++ ){
          var routerId = theRouter[i].id//放入路由器的id,便于遍历边
          var routerLable = theRouter[i].label//放入路由器的id,便于遍历边
          var isDelete = theRouter[i].delete//放入路由器的id,便于遍历边
          let routingTable = new Array();//路由表(单表)
          for(let j = 0; j < theNetwork.length; j++ ){
            let target = {targetName:theNetwork[j].label,next:'-',distance:16,id:theNetwork[j].id }//one row of ratingtable
            routingTable.push(target)
          }
          // console.log(routingTable)
          routingTables.push({routerId,routerLable,isDelete,routingTable});
        }
      }
      var oldNetwork = this.theNetwork
      if(oldNetwork.length>theNetwork.length){
        for(let  i=0;i<oldNetwork.length;i++){
          let flag= true
          for(let j=0;j<theNetwork.length;j++){
            if(oldNetwork[i].id == theNetwork[j].id){
              flag =false
              break
            }
          }
          oldNetwork[i].delete = flag
        }
      }
      else {
        for(let r =0 ;r<theNetwork.length-oldNetwork.length&&!hasadd;r++){
          for(let j = 0; j < theRouter.length; j++ ){
            let target = {targetName:theNetwork[oldNetwork.length+r].label,next:'-',distance:16,id:theNetwork[oldNetwork.length+r].id }//one row of ratingtable
            routingTables[j].routingTable.push(target);
          }

        }
      oldNetwork = JSON.parse(JSON.stringify(theNetwork))
      }
      console.log(oldNetwork)
      var oldRouter = this.theRouter
      if(oldRouter.length>theRouter.length){
        for(let  i=0;i<oldRouter.length;i++){
          let flag= true
          for(let j=0;j<theRouter.length;j++){
            if(oldRouter[i].id == theRouter[j].id){
              flag =false
              break
            }
          }
          oldRouter[i].delete = flag
          routingTables[i].isDelete = flag
        }
      }
      else {
        for(let r =0 ;r<theRouter.length-oldRouter.length&&JSON.stringify(oldRouter)!='[]';r++){
          var routingTable = [  ]
          for(let j = 0; j < theNetwork.length; j++ ){
            let target = {targetName:theNetwork[j].label,next:'-',distance:16,id:theNetwork[j].id }//one row of ratingtable
            routingTable.push(target)
          }
          console.log(theRouter[oldRouter.length+r])
          var routerId = theRouter[oldRouter.length+r].id
          var routerLable = theRouter[oldRouter.length+r].label
          var isDelete = theRouter[oldRouter.length+r].delete
          routingTables.push({routerId,routerLable,isDelete,routingTable});
        }
      oldRouter = JSON.parse(JSON.stringify(theRouter))
      }
      console.log(oldRouter)

      //导入边
      var edgesTable = []
      for(let i=0;i<edges.length;i++){//遍历
        var edge = {from:edges[i].source,to:edges[i].target}
        edgesTable.push(edge)
      }
      console.log((routingTables))
      for(let  i=0;i<edgesTable.length;i++){
        for(let j=0;j<routingTables.length;j++){
          if(routingTables[j].routerId == edgesTable[i].from || routingTables[j].routerId == edgesTable[i].to){
            if(oldRouter[j].error || oldRouter[j].delete){
              for(let k=0;k<routingTables[j].routingTable.length;k++){
                routingTables[j].routingTable[k].next = '-'
                routingTables[j].routingTable[k].distance = 16
              }
            }
            else{
              for(let k=0;k<routingTables[j].routingTable.length;k++){
                if(routingTables[j].routingTable[k].id == edgesTable[i].from || routingTables[j].routingTable[k].id == edgesTable[i].to){
                  routingTables[j].routingTable[k].distance = 1
                  // console.log(routingTables[j].routerId)
                  // console.log(routingTables[j].routingTable[k])
                }
              }
            }
          }
        }
      }
      // console.log(routingTables)
      this.routingTables = JSON.parse(JSON.stringify(routingTables))
      var theShowRoutingTables = []
      for(let n=0;n<routingTables.length;n++){
        if(!(routingTables[n].isDelete)){
          theShowRoutingTables.push(routingTables[n])
        }
      }
      this.theShowRoutingTables = JSON.parse(JSON.stringify(theShowRoutingTables))
      this.theRipGraph = JSON.parse(JSON.stringify(theRipGraph))
      this.theRouter = JSON.parse(JSON.stringify(oldRouter))
      this.theNetwork = JSON.parse(JSON.stringify(oldNetwork))
      this.edgesTable = JSON.parse(JSON.stringify(edgesTable))

    },
    checkpath(from,to){
      // console.log(this.getlable(from)+'check'+this.getlable(to))
     var routingTables = this.routingTables
      for(let i=0;i<routingTables.length;i++){
        if(routingTables[i].routerId == from){
          for(let j=0;j<routingTables[i].routingTable.length;j++){
            if(routingTables[i].routingTable[j].id == to){
              // console.log(routingTables[i].routingTable[j].distance)
              if(routingTables[i].routingTable[j].distance == 1){
              // console.log(this.getlable(from)+'ok'+this.getlable(to))
                
                return true
              }

              else if(routingTables[i].routingTable[j].distance < 16){
                // console.log(this.getlable(from)+'mid'+this.getlable(to))
                for(let k=0;k<routingTables.length;k++){

                  if(routingTables[k].routerLable == routingTables[i].routingTable[j].next){
                    return this.checkpath(routingTables[k].routerId,to)
                  }
                }
              }
              else if(routingTables[i].routingTable[j].distance >= 16){
                                // console.log(this.getlable(from)+'no'+this.getlable(to))
                return false

              }
            }
          }
        }
      }
    },
    getlable(node){
      var nodes = this.graph.save().nodes
      for(let i=0;i<nodes.length;i++){
        if(nodes[i].id==node){
          return nodes[i].label
        }
      }
    }
  }
};
</script>
<style>
.name{
	text-align: center;
	background-color: #409EFF;
	color: rgb(255, 255, 255);
	height: 20px;
	line-height: 20px;
	font-size: 1rem;
	font-weight: bold;
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
        transform: scale(1.05);
        box-shadow: 6px 10px 10px rgba(194, 194, 194, 0.24);
    }    
</style>
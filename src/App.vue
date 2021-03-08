<template>
  <div id="app">
    <G6Editor class="g6" mode="edit">
    </G6Editor>
<RoutingTable></RoutingTable>
  </div>
</template>

<script>
import G6Editor from './components/G6Editor'
import RoutingTable from './components/RoutingTable'
export default {
  name: 'app',
  components:{G6Editor,RoutingTable}
}
</script>

<style>
html, body {
    margin: 0;
    font-size: 12px;
}
#app{
  margin: 0;
  padding: 0;
  height: fit-content;
}

</style>

<template>
  <div class="row"
    @mousedown="$emit('rowMouseDown')"
    @mouseover="$emit('rowMouseOver')"
  >
    <InfiniteTableCell v-for="(cell, iCol) of rowData"
    :cellData="cell"
    @cellMouseDown="$emit('cellMouseDown', iCol)"
    @cellMouseOver="$emit('cellMouseOver', iCol)"
    :key="iCol"/>
  </div>
</template>

<script>
import InfiniteTableCell from './InfiniteTableCell.vue'

export default {
  name: 'InfiniteTableRow',
  components:{
    InfiniteTableCell
  },
  props: ['rowData']
}
</script>

<style scoped>
.row{
  display:flex;
  line-height:30px;
}
</style>

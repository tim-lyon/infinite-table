<template>
  <div class="row">
    <InfiniteTableCell
      v-for="(cell, columnIndex) of data"
      :config="config"
      :value="cell"
      @cellMouseDown="$emit('cellMouseDown', {R: rowIndex, C: columnIndex})"
      @cellMouseOver="$emit('cellMouseOver', {R: rowIndex, C: columnIndex})"
      :key="columnIndex"
      :selected-range="selectedRange"
      :is-active="rowIndex === activeCell.R && columnIndex === activeCell.C"
      :row-index="rowIndex"
      :column-index="columnIndex"
    />
  </div>
</template>

<script>
import InfiniteTableCell from './InfiniteTableCell.vue'

export default {
  name: 'InfiniteTableRow',
  components: {
    InfiniteTableCell
  },
  props: ['config', 'data', 'rowIndex', 'selectedRange', 'activeCell'],
  created(){
    this.$emit('created')
  }
}
</script>

<style scoped>
.row{
  display:flex;
  line-height:30px;
}
</style>

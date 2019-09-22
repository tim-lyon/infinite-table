<template>
  <tr class="row" :style="rowStyle">
    <td is="InfiniteTableCell"
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
  </tr>
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
  },
  computed: {
    rowStyle() {
      let background = 'none'
      if(this.config.style && this.config.style.row){
        if(this.config.style.row.background){
          let color = this.config.style.row.background
          background = Array.isArray(color) ? color[this.rowIndex%color.length] : color
        }
      }

      const border = this.config.style.row.border
      return {
        background: background,
        borderTop: border.width + 'px solid ' + border.color,
        borderBottom: border.width + 'px solid ' + border.color,
        lineHeight: '30px'
      }
    }
  }
}
</script>

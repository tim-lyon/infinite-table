<template>
  <table>
    <tr v-for="row of rowCount" :key="row">
      <td
        v-for="(cell, index) of headerCells[row-1]"
        :key="index"
        :colspan="cell.colspan"
        :rowspan="cell.rowspan"
        :class="{selected: isColumnSelected(cell), active: isColumnActive(cell)}"
        @mousedown.stop="onColumnMouseDown({start: cell.column, end:cell.column + cell.colspan-1})"
        @mouseover.stop="onColumnMouseOver({start: cell.column, end:cell.column + cell.colspan-1})"
      >
        {{cell.name}}
      </td>
    </tr>
  </table>
</template>

<script>

var mouseDownColumns

export default {
  name: 'InfiniteTableHeaders',
  props: {
    headers: Array,
    selectedColumns: Object,
    allRowsSelected: Boolean,
  },
  computed: {
    headerCells() {
      var headers = {}
      this.getHeaderRow(this.headers, 0, 0, headers)
      const rowCount = Object.keys(headers).length
      for(var row = 0; row < rowCount; row++){
        for(var cell of headers[row]){
          cell.rowspan = 1 + rowCount - cell.depth - row
          cell.rowspan = cell.depth == 1 ? (rowCount - row) : 1
        }
      }
      return headers
    },
    rowCount(){
      return Object.keys(this.headerCells).length
    }
  },
  methods: {
    getHeaderRow(header, row, column, result){
      for(const h of header){
        if(!result.hasOwnProperty(row)){
          result[row] = []
        }
        const colspan = this.numberOfColumns(h)
        result[row].push({
          name: h.name,
          type: h.type,
          column: column,
          colspan: colspan,
          depth: this.depthOf(h)
        })
        if(h.hasOwnProperty('children')){
          this.getHeaderRow(h.children, row+1, column, result)
        }
        column += colspan
      }
    },
    depthOf(header){
      let depth = 1
      if(header.hasOwnProperty('children')){
        depth += header.children.reduce((a, b) => Math.max(this.depthOf(a), this.depthOf(b)))
      }
      return depth
    },
    selectColumnRange(range){
      this.$emit('selectColumnRange', range)
    },
    onColumnMouseDown(range){
      mouseDownColumns = range
      this.selectColumnRange(range)
    },
    onColumnMouseOver(range){
      if(event.buttons != 1 || !mouseDownColumns) return
      if(range.start > mouseDownColumns.start){
        range.start = mouseDownColumns.start
      }
      if(range.end < mouseDownColumns.end){
        range.end = mouseDownColumns.end
      }
      this.$emit('selectColumnRange', range)
    },
    onMouseUp(){
      mouseDownColumns = false
    },
    numberOfColumns(header){
      if(header.hasOwnProperty('children')){
        return header.children.reduce((count, h) => count + this.numberOfColumns(h), 0);
      }
      return 1
    },
    isColumnActive(cell){
      if(!this.selectedColumns) return false
      return cell.column >= this.selectedColumns.start &&
      (cell.column + cell.colspan - 1) <= this.selectedColumns.end
    },
    isColumnSelected(cell){
      if(!this.allRowsSelected || !this.selectedColumns) return false
      return cell.column >= this.selectedColumns.start &&
      (cell.column + cell.colspan - 1) <= this.selectedColumns.end
    }
  }
}
</script>

<style scoped>
.active{
  //background:rgba(0, 135, 189, .1);
}
.selected{
  //background:#aaa;
  //border: 1px solid #aaa;
  //border-bottom:none;
}
table {
  border-collapse: collapse;
}
td {
  color:rgba(0,0,0,0.54);
  min-width: 6em;
  width: 6em;
  border:1px solid #aaa;
  border-bottom:none;
  text-align: center;
  line-height: 1.3em;
  padding: 0;
  cursor: cell;
}
</style>

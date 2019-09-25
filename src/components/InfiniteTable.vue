<template>
<div>
  <div class="table-container" tabindex="1" @keydown="OnTableKeypress">
    <InfiniteTableHeaders
      :config="config"
      :headers="computedHeaders"
      :selectedColumns="selectedColumns"
      @selectColumnRange="selectColumns"
    />
    <div class="table-body" ref="body" :style="{height:contentHeight+'px'}" @scroll="scrollTable">
      <div :style="{height: totalHeight+'px'}">
        <table :style="{position:'relative', top:topBufferHeight+'px', borderCollapse: 'collapse'}">
          <colgroup>
            <col v-for="i of columnCount" :key="i" :style="columnStyle"/>
          </colgroup>
          <tr is="InfiniteTableRow"
          v-for="i in renderedRowCount"
          :config="config"
          :row-index="i+startRow"
          @input="setData($event, i+startRow)"
          @cellMouseDown="onCellMouseDown"
          @cellMouseOver="onCellMouseOver"
          :data="rowData(i+startRow)"
          :selected-range="selectedRange"
          :active-cell="active"
          :key="i+startRow" />
        </table>
      </div>
    </div>
  </div>
  <div>
    <button v-for="(exportFunction, name) in computedExports" :key="name" @click="startExport(exportFunction)">
      Export {{name}}
    </button>
  </div>

  <br><br>Selection: {{selection}}
  <br>Active: {{active}}
  <br>Editing: {{isEditing}}
  <br>v: {{inputValue}}
</div>
</template>

<script>
import InfiniteTableHeaders from './InfiniteTableHeaders.vue'
import InfiniteTableRow from './InfiniteTableRow.vue'

const ROW_HEIGHT = 30
const BUFFER_ROWS = 5
const ROW_COUNT = 12
const MAX_ROWS = 100000
const DEFAULT_CONFIG = {
  style: {
    row: {
      border: {
        color: '#aaa',
        width: 1
      }
    },
    column: {
      border: {
        color: '#aaa',
        width: 1
      }
    },
    selection: {
      border: {
        color: 'rgb(0, 135, 189)',
        width: 1
      },
      fill: {
        color: 'rgba(0, 135, 189, .13)'
      }
    }
  }
}
export default {
  name: 'InfiniteTable',
  components: {
    InfiniteTableHeaders,
    InfiniteTableRow
  },
  props: {
    config: {
      type: Object,
      required: false,
      default() {
        return DEFAULT_CONFIG
      }
    },
    headers: {
      required: false
    },
    tableData: {
      type: Array,
      required: true
    },
    exports: {
      required: false
    }
  },
  methods: {
    rowData(iRow) {
      return this.tableData[iRow]
    },
    startExport(exportFunction){
      exportFunction()
    },
    exportCSV(){
      let csvFile = "data:text/csv;charset=utf-8,";
      for(var iRow = 1; iRow <= this.rowCount; iRow++){
        const rowArray = this.rowData[iRow].map(fieldValue => {
          var value = fieldValue.hasOwnProperty('value') ? fieldValue.value : fieldValue
          var string = value.toString().replace(/"/g, '""')
          if (string.search(/("|,|\n)/g) >= 0){
            string = '"' + string + '"'
          }
          return string
        })
        csvFile += rowArray.join(",") + "\r\n";
      }
      var blob = new Blob([csvFile], { type: 'text/csv;charset=utf-8;' });
      var url = URL.createObjectURL(blob);
      var link = document.createElement("a");
      link.setAttribute("href", url);
      link.setAttribute("download", "export.csv");
      link.style.visibility = 'hidden';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    cellRange(row, column){
      return {
        start: {
          R: row,
          C: column,
        },
        end: {
          R: row,
          C: column,
        }
      }
    },
    setData(data, row){
        this.$emit('input', {
          range: this.cellRange(row, data.column),
          value: data.value
        })
    },
    defaultColumnName(iCol) {
      for (var ret = '', a = 1, b = 26; (iCol -= a) >= 0; a = b, b *= 26) {
        ret = String.fromCharCode(parseInt((iCol % b) / a) + 65) + ret;
      }
      return ret;
    },
    OnTableKeypress(){
      switch(event.key){
        case 'ArrowDown':
          this.OnArrow(0,1)
          break
        case 'ArrowUp':
          this.OnArrow(0,-1)
          break
        case 'ArrowLeft':
          this.OnArrow(-1,0)
          break
        case 'ArrowRight':
          this.OnArrow(1,0)
          break
        case 'Tab':
          event.preventDefault()
          this.OnTab()
          break
        case 'Enter':
          event.preventDefault()
          this.OnEnter()
          break
        case 'PageUp':
          this.OnArrow(0,-ROW_COUNT)
          break
        case 'PageDown':
          this.OnArrow(0,ROW_COUNT)
          break
        default:
          this.isEditing = true
      }
    },
    scrollToRow(iRow){
      const distanceFromReferenceRow = this.$refs.body.scrollTop-this.referenceRow.scrollTop
      const rowsFromReferenceRow = distanceFromReferenceRow/ROW_HEIGHT
      const firstFullyVisibleRow = Math.ceil(this.referenceRow.row+rowsFromReferenceRow)+1
      const offset = iRow < firstFullyVisibleRow ? 1 :
      iRow > firstFullyVisibleRow + ROW_COUNT - 2 ? ROW_COUNT : 0
      if(offset!=0){
        this.$refs.body.scrollTop = this.referenceRow.scrollTop +
                (iRow-this.referenceRow.row-offset)*ROW_HEIGHT
      }
    },
    OnArrow(dx, dy){
      if(event.shiftKey){
        this.selection.end.C += dx
        this.selection.end.R += dy
      }else{
        this.selection.start.C += dx
        this.selection.start.R += dy
        this.selection.end.C = this.selection.start.C
        this.selection.end.R = this.selection.start.R
        this.setActiveRow(this.selection.start.R)
        this.setActiveColumn(this.selection.start.C)
      }
      this.clampSelection()
      this.scrollToRow(this.selection.end.R)
    },
    OnTab(){
      if(this.selection.start.R === this.selection.end.R &&
      this.selection.start.C === this.selection.end.C){
        this.OnArrow(1,0)
        return
      }
      const newRow = this.active.C === this.selectedRange.end.C
      this.setActiveColumn( newRow ? this.selectedRange.start.C : this.active.C + 1)
      if(newRow){
        this.setActiveRow(this.active.R === this.selectedRange.end.R ?
        this.selectedRange.start.R : this.active.R + 1)
      }
    },
    OnEnter(){
      if(this.selection.start.R === this.selection.end.R &&
      this.selection.start.C === this.selection.end.C){
        this.OnArrow(0,1)
        return
      }
      const newColumn = this.active.R === this.selectedRange.end.R
      this.setActiveRow( newColumn ? this.selectedRange.start.R : this.active.R + 1)
      if(newColumn){
        this.setActiveColumn(this.active.C === this.selectedRange.end.C ?
        this.selectedRange.start.C : this.active.C + 1)
      }
    },
    clampSelection(){
      if(this.selection.start.R < 0){
        this.selection.start.R = 0
      }
      else if(this.selection.start.R >= this.rowCount){
        this.selection.start.R = this.rowCount - 1
      }

      if(this.selection.end.R < 0){
        this.selection.end.R = 0
      }
      else if(this.selection.end.R >= this.rowCount){
        this.selection.end.R = this.rowCount - 1
      }

      if(this.selection.start.C < 0){
        this.selection.start.C = 0
      }
      else if(this.selection.start.C >= this.columnCount){
        this.selection.start.C = this.columnCount - 1
      }

      if(this.selection.end.C < 0){
        this.selection.end.C = 0
      }
      else if(this.selection.end.C >= this.columnCount){
        this.selection.end.C = this.columnCount - 1
      }
    },
    selectColumns(columns){
      if(!event.shiftKey){
        this.selection.start.C = columns.start - 1
      }
      this.selection.end.C = columns.end - 1
      this.selection.start.R = 0
      this.selection.end.R = this.rowCount - 1
    },
    scrollTable(){
      const scrollTop = event.target.scrollTop
      if(this.rowCount > MAX_ROWS){
        const step = scrollTop - this.lastScrollTop
        if(Math.abs(step) > 0.01*MAX_ROWS){
          if(scrollTop < MAX_ROWS){
            //align with top
            this.referenceRow.row = 0
            this.referenceRow.scrollTop = 0
          }
          else if(event.target.scrollHeight-scrollTop < MAX_ROWS){
            // align with bottom
            this.referenceRow.row = this.rowCount
            this.referenceRow.scrollTop = event.target.scrollHeight
          }
          else{
            // align based on percentage
            this.referenceRow.row = Math.round(this.rowCount*scrollTop/event.target.scrollHeight)
            this.referenceRow.scrollTop = scrollTop
          }
        }

        const distanceFromReferenceRow = scrollTop - this.referenceRow.scrollTop
        const rowsFromReferenceRow = Math.round(distanceFromReferenceRow/30)
        const startRow = this.referenceRow.row + rowsFromReferenceRow
        this.startRow = Math.max(startRow - BUFFER_ROWS, 0)
        const lastRow = Math.min(this.startRow + ROW_COUNT+2*BUFFER_ROWS, this.rowCount)
        this.renderedRows = lastRow - this.startRow
        this.topBufferHeight = this.referenceRow.scrollTop + 30*rowsFromReferenceRow
        this.topBufferHeight -= 30*(startRow - this.startRow)
      }else{
        let firstVisibleRow = Math.round(scrollTop/30)
        this.startRow = Math.max(firstVisibleRow - BUFFER_ROWS, 0)
        const lastRow = Math.min(this.startRow + ROW_COUNT+2*BUFFER_ROWS, this.rowCount)
        this.renderedRows = lastRow - this.startRow
        this.topBufferHeight = this.startRow*ROW_HEIGHT
      }
      this.lastScrollTop = scrollTop
    },
    onCellMouseDown(cell){
      if(this.isEditing && cell.C !== this.active.C){
        this.isEditing = false
      }
      this.selection.end.C = cell.C
      if(!event.shiftKey){
        this.selection.start.C = cell.C
        this.setActiveColumn(cell.C)
      }

      this.selection.end.R = cell.R
      if(!event.shiftKey){
        this.selection.start.R = cell.R
        this.setActiveRow(cell.R)
      }
      window.getSelection().removeAllRanges();
    },
    onCellMouseOver(cell){
      if(event.buttons == 1){
        this.selection.end.C = cell.C
        this.selection.end.R = cell.R
      }
    },
    setActiveColumn(iCol){
      if(this.isEditing && iCol !== this.active.C){
        this.isEditing = false
      }
      if(iCol >= 0 && iCol < this.columnCount){
        this.active.C = iCol
      }
    },
    setActiveRow(iRow){
      if(this.isEditing && iRow !== this.active.R){
        this.isEditing = false
      }
      if(iRow >= 0 && iRow < this.rowCount){
        this.active.R = iRow
      }
    }
  },
  data(){
    return {
      referenceRow: {row: 0, scrollTop: 0},
      lastScrollTop: 0,
      renderedRows: ROW_COUNT+2*BUFFER_ROWS,
      startRow: -1,
      topBufferHeight: 0,
      contentHeight: ROW_COUNT*ROW_HEIGHT,

      isEditing: false,
      active: {
        R: 0,
        C: 0
      },
      selection: {
        start: {
          R:0,
          C:0
        },
        end: {
          R:0,
          C:0
        }
      }
    }
  },
  computed: {
    columnStyle(){
      const border = this.config.style.column.border
      return {
        borderWidth: '0 ' + border.width + 'px',
        borderStyle: 'solid',
        borderColor: border.color
      }
    },
    inputValue() {
      return this.tableData[this.active.R][this.active.C]
    },
    selectedRange(){
      return {
        start: {
          R: Math.min(this.selection.start.R, this.selection.end.R),
          C: Math.min(this.selection.start.C, this.selection.end.C)
        },
        end: {
          R: Math.max(this.selection.start.R, this.selection.end.R),
          C: Math.max(this.selection.start.C, this.selection.end.C)
        }
      }
    },
    computedHeaders() {
      if(Array.isArray(this.headers) || this.headers === undefined){
        return this.headers
      }
      return this.tableData[0].map((cell, iCol) => {
        return {name: this.defaultColumnName(iCol+1)}
      })
    },
    computedExports(){
      return Array.isArray(this.exports) || this.exports === undefined ?
      this.exports : {CSV: this.exportCSV}
    },
    rowCount() {
      return this.tableData.length
    },
    columnCount() {
      return this.tableData[0].length
    },
    renderedRowCount() {
      return Math.min(this.renderedRows, this.rowCount - this.startRow)
    },
    totalHeight() {
      return Math.min(MAX_ROWS, this.rowCount)*ROW_HEIGHT
    },
    selectedColumns(){
      if(Math.abs(this.selection.start.R -
      this.selection.end.R) == this.rowCount-1){
        return {
          start: this.selectedRange.start.C,
          end: this.selectedRange.end.C
        }
      }
      return null
    }
  }
}
</script>

<style scoped>
.table-container{
  border-bottom:1px solid lime;
  outline:none;
}
.table-body{
  overflow-y: scroll;
  position: relative;
  cursor: cell;
}
.the-input{
  display: block;
  height: 100%;
  width: 100%;
  outline: none;
  border: none;
  background: none;
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
}
</style>

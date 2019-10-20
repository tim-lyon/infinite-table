<template>
<div class="table-container" tabindex="1" @keydown.self="OnTableKeypress">
  <InfiniteTableHeaders
    style="margin-left:51px;"
    :headers="computedHeaders"
    :selectedColumns="selectedColumns"
    :allRowsSelected="allRowsSelected"
    @selectColumnRange="selectColumns"
  />
  <div class="table-body" ref="body" :style="{height:contentHeight+'px'}" @scroll="scrollTable">
    <div :style="{height: totalHeight+'px'}">
      <table :style="{position:'relative', top:topBufferHeight+'px', borderCollapse: 'collapse'}">
        <colgroup>
          <col v-for="i of columnCount" :key="i"/>
        </colgroup>
        <tr
        v-for="row in renderedRowArray"
        :row-index="row"
        :key="row - startRow" >
          <th :class="{rowId: true, active: row >= selectedRows.start && row <= selectedRows.end, selected: allColumnsSelected }"
            @mousedown="onCellMouseDown({R: row, C: -1}, $event)"
            @mouseover="onCellMouseOver({R: row, C: -1}, $event)"
          >
            {{row}}
            </th>
          <td is="InfiniteTableCell"
            v-for="column in columnCount"
            :key="column"
            :selected-range="selectedRange"
            :active-cell="active"
            :row-index="row"
            :column-index="column -1"
            @mousedown="onCellMouseDown({R: row, C: column -1}, $event)"
            @mouseover="onCellMouseOver({R: row, C: column -1}, $event)"
            @dblclick="onCellDblClick({R: row, C: column -1})"
          >
            <input v-if="isEditing && active.R == row && active.C == column - 1"
              type="text"
              class="the-input"
              v-model="inputValue"
              ref="theinput"
              @keydown="onInputKeypress"
            >
            <template v-else class="the-input">{{getCellValue(row, column -1)}}</template>
          </td>
        </tr>
      </table>
    </div>
  </div>
</div>
</template>

<script>
import InfiniteTableHeaders from './InfiniteTableHeaders.vue'
import InfiniteTableCell from './InfiniteTableCell.vue'

const ROW_HEIGHT = 30
const BUFFER_ROWS = 5
const ROW_COUNT = 12
const MAX_ROWS = 100000

const clamp = (value, min, max) => value < min ? min : value > max ? max : value

function CellReference() {
  this.R = -1
  this.C = -1
}

export default {
  name: 'InfiniteTable',
  components: {
    InfiniteTableHeaders,
    InfiniteTableCell
  },
  model: {
    prop: 'tableData',
    event: 'update'
  },
  props: {
    columnCount: {
      type: Number,
      required: true
    },
    rowCount: {
      type: Number,
      required: true
    },
    headers: {
      required: false,
      default: true
    },
    selectable: {
      type: Boolean,
      required: false,
      default: true
    },
    editable: {
      type: Boolean,
      required: false,
      default: true
    },
    tableData: {
      type: Object,
      required: true
    }
  },
  data(){
    return {
      referenceRow: {row: 0, scrollTop: 0},
      lastScrollTop: 0,
      renderedRows: ROW_COUNT+2*BUFFER_ROWS,
      startRow: 0,
      topBufferHeight: 0,
      isEditing: false,
      isDeepEditing: false,
      isSelectable: true,
      isEditable: true,
      inputValue: '',
      isEndKeyPressed: false,
      active: new CellReference(),
      selection: {
        start: new CellReference(),
        end: new CellReference()
      }
    }
  },
  created(){
    this.hasGetFunction = this.tableData.hasOwnProperty('get')
    this.hasSetFunction = this.tableData.hasOwnProperty('set')
    this.isEditable = this.editable
    this.isSelectable = this.selectable || this.editable
  },
  methods: {
    getCellValue(row, column){
      return this.hasGetFunction ? this.tableData.get(row, column) : this.tableData[row][column]
    },
    setRangeValue(range, value){
      if(this.hasSetFunction){
        this.tableData.set(range, value)
      }else{
        for(var row = range.start.R; row <= range.end.R; row++){
          for(var column = range.start.C; column <= range.end.C; column++){
            this.$set(this.tableData[row], column, value)
          }
        }
      }
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
    defaultColumnName(iCol) {
      for (var ret = '', a = 1, b = 26; (iCol -= a) >= 0; a = b, b *= 26) {
        ret = String.fromCharCode(parseInt((iCol % b) / a) + 65) + ret;
      }
      return ret;
    },
    OnTableKeypress(event){
      const rowShift = this.isEndKeyPressed ? this.rowCount : 1
      const columnShift = (this.isEndKeyPressed || event.key == 'Home') ? this.columnCount : 1
      switch(event.key){
        case 'Escape':
          this.endEdit()
          this.OnArrow(event, 0, 0)
          break
        case 'ArrowDown':
          event.preventDefault() // stop window scrolling
          this.OnArrow(event, 0, rowShift)
          break
        case 'ArrowUp':
          event.preventDefault() // stop window scrolling
          this.OnArrow(event, 0, -rowShift)
          break
        case 'ArrowLeft':
        case 'Home':
          event.preventDefault() // stop window scrolling
          this.OnArrow(event, -columnShift, 0)
          break
        case 'ArrowRight':
          event.preventDefault() // stop window scrolling
          this.OnArrow(event, columnShift, 0)
          break
        case 'Tab':
          event.preventDefault() // stop table losing focus
          this.OnTab(event)
          break
        case 'Enter':
          event.preventDefault() // stop window scrolling
          this.OnEnter(event)
          break
        case 'PageUp':
          event.preventDefault() // stop window scrolling
          this.OnArrow(event, 0,-ROW_COUNT)
          break
        case 'PageDown':
          event.preventDefault() // stop window scrolling
          this.OnArrow(event, 0,ROW_COUNT)
          break
        case 'Delete':
          if(this.isEditing){
            return
          }
          this.OnDelete()
          break;
        case 'Backspace':
          if(this.isEditing){
            return
          }
          this.OnDelete()
          this.startEdit()
          break
        case 'F2':
          this.startEdit()
          break
        case 'a':
          if(event.ctrlKey){
            this.selectAll()
            break
          }
        default:
          if(!this.isEditing && event.key.length == 1) {
            this.startEdit(true)
          }
      }
      this.isEndKeyPressed = event.key == 'End'
      if(this.isEndKeyPressed){
        event.preventDefault() // stop window scrolling
      }
    },
    onInputKeypress(event){
      switch(event.key){
        case 'Escape':
          this.cancelEdit()
          this.OnArrow(0, 0)
          break
        case 'ArrowLeft':
        case 'ArrowRight':
          if(this.isDeepEditing){
            return
          }
        default:
          this.OnTableKeypress(event)
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
    OnArrow(event, dColumn, dRow, ignoreShiftKey){
      if(event.shiftKey && !ignoreShiftKey){
        this.selection.end.C += dColumn
        this.selection.end.R += dRow
      }else{
        this.setActiveRow(this.active.R + dRow)
        this.setActiveColumn(this.active.C + dColumn)
        this.selection.start.R = this.active.R
        this.selection.end.R = this.active.R
        this.selection.start.C = this.active.C
        this.selection.end.C = this.active.C
      }
      this.clampSelection()
      this.scrollToRow(this.selection.end.R)
    },
    OnTab(event){
      if(this.selectedCellCount == 1){
        this.OnArrow(event, event.shiftKey ? -1 : 1, 0, true)
        return
      }
      const last = event.shiftKey ? this.selectedRange.start : this.selectedRange.end
      const first = event.shiftKey ? this.selectedRange.end : this.selectedRange.start
      const newRow = this.active.C === last.C
      this.setActiveColumn( newRow ? first.C : this.active.C + (event.shiftKey ? -1 : 1))
      if(newRow){
        this.setActiveRow(this.active.R === last.R ?
        first.R : this.active.R + (event.shiftKey ? -1 : 1))
      }
    },
    OnEnter(event){
      if(this.isEditing){
        this.endEdit(event.ctrlKey)
      }
      if(this.selectedCellCount == 1){
        this.OnArrow(event, 0, event.shiftKey ? -1 : 1, true)
        return
      }
      const last = event.shiftKey ? this.selectedRange.start : this.selectedRange.end
      const first = event.shiftKey ? this.selectedRange.end : this.selectedRange.start
      const newColumn = this.active.R === last.R
      this.setActiveRow( newColumn ? first.R : this.active.R + (event.shiftKey ? -1 : 1))
      if(newColumn){
        this.setActiveColumn(this.active.C === last.C ?
        first.C : this.active.C + (event.shiftKey ? -1 : 1))
      }
    },
    selectAll(){
      this.selection.start.R = 0
      this.selection.end.R = this.rowCount - 1
      this.selection.start.C = 0
      this.selection.end.C = this.columnCount - 1
    },
    clampSelection(){
      this.selection.start.R = clamp(this.selection.start.R, 0, this.rowCount - 1)
      this.selection.end.R = clamp(this.selection.end.R, 0, this.rowCount - 1)
      this.selection.start.C = clamp(this.selection.start.C, 0, this.columnCount - 1)
      this.selection.end.C = clamp(this.selection.end.C, 0, this.columnCount - 1)
      console.log(this.selection.start.R)
    },
    selectColumns(columns){
      this.active.R = 0
      this.active.C = columns.start
      if(!event.shiftKey){
        this.selection.start.C = columns.start
      }
      this.selection.end.C = columns.end
      this.selection.start.R = 0
      this.selection.end.R = this.rowCount - 1
    },
    scrollTable(event){
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
    onCellMouseDown(cell, event){
      if(this.isEditing) {
        if(cell.C === this.active.C && cell.R === this.active.R){
          this.isDeepEditing = true
        }
        else {
          this.endEdit()
        }
      }

      if(cell.C == -1){
        this.selection.start.C = 0
        this.selection.end.C = this.columnCount - 1
      }else{
        this.selection.end.C = cell.C
        if(!event.shiftKey){
          this.selection.start.C = cell.C
          this.setActiveColumn(cell.C)
        }
      }

      this.selection.end.R = cell.R
      if(!event.shiftKey){
        this.selection.start.R = cell.R
        this.setActiveRow(cell.R)
      }
      window.getSelection().removeAllRanges();
    },
    onCellMouseOver(cell, event){
      if(event.buttons == 1){
        if(cell.C >= 0){
          this.selection.end.C = cell.C
        }
        this.selection.end.R = cell.R
      }
    },
    onCellDblClick(cell) {
      this.startEdit()
    },
    setActiveColumn(iCol){
      if(this.isEditing && iCol !== this.active.C){
        this.endEdit()
      }
      this.active.C = clamp(iCol, 0, this.columnCount - 1)
    },
    setActiveRow(iRow){
      if(this.isEditing && iRow !== this.active.R){
        this.endEdit()
      }
      this.active.R = clamp(iRow, 0, this.rowCount - 1)
    },
    OnDelete(){
      if(this.isEditable){
        this.setRangeValue(this.selectedRange, null)
      }
    },
    startEdit(resetValue) {
      if(!this.isEditable) return
      this.inputValue = resetValue ? '' : this.getCellValue(this.active.R, this.active.C)
      this.isEditing = true
      this.isDeepEditing = !resetValue
      this.$nextTick(()=>
        this.$refs.theinput[0].focus()
      )
    },
    endEdit(fillSelection) {
      if(!this.isEditing){
        return
      }
      if(fillSelection) {
        this.setRangeValue(this.selectedRange, this.inputValue)
      }
      else {
        this.setRangeValue(this.cellRange(this.active.R, this.active.C), this.inputValue)
      }
      this.cancelEdit()
    },
    cancelEdit(){
      this.isEditing = false
      this.isDeepEditing = false
      this.$el.focus()
    }
  },
  computed: {
    contentHeight() {
      return Math.min(ROW_COUNT, this.rowCount)*ROW_HEIGHT
    },
    selectedRange(){
      if(!this.isSelectable){
        return {
          start: new CellReference(),
          end: new CellReference()
        }
      }
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
    selectedCellCount(){
      return (1 + this.selectedRange.end.C - this.selectedRange.start.C) *
      (1 + this.selectedRange.end.R - this.selectedRange.start.R)
    },
    computedHeaders() {
      if(Array.isArray(this.headers) || this.headers == false || this.headers == 'false'){
        return this.headers
      }
      let headers = []
      for(var column = 1; column <= this.columnCount; column++){
        headers.push({name: this.defaultColumnName(column)})
      }
      return headers
    },
    renderedRowCount() {
      return Math.min(this.renderedRows, this.rowCount - this.startRow)
    },
    renderedRowArray() {
      let renderedRows = []
      var currentRow = this.startRow
      for(var i = 0; i < this.renderedRowCount; i ++){
        renderedRows.push(currentRow++)
      }
      return renderedRows
    },
    totalHeight() {
      return Math.min(MAX_ROWS, this.rowCount)*ROW_HEIGHT
    },
    allRowsSelected() {
      return this.selection.start.R == 0 &&
        this.selection.end.R  == this.rowCount-1
    },
    allColumnsSelected() {
      return this.selection.start.C == 0 &&
        this.selection.end.C  == this.columnCount-1
    },
    selectedColumns(){
      return {
        start: this.selectedRange.start.C,
        end: this.selectedRange.end.C
      }
    },
    selectedRows(){
      return {
        start: this.selectedRange.start.R,
        end: this.selectedRange.end.R
      }
    }
  }
}
</script>

<style scoped>
.table-container{
  border-bottom:1px solid #aaa;
  
  outline:none;
  display: table;
}
.table-body{
  border-top: 1px solid #aaa;
  border-right:1px solid #aaa;
  overflow-y: scroll;
  position: relative;
  cursor: cell;
}

.the-input{
  display: block;
  outline: none;
  border: none;
  padding:0;
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
}
col {
  border: 1px solid #aaa;
  border-top: 1px solid white;;
}
tr {
  background: none;
  border: 1px solid #aaa;
  border-top-width: 0px;
}
td {
  margin: auto;
}
th {
  color:rgba(0,0,0,0.54);
  font-weight: inherit;
}

.rowId{
  width: 48px;
  min-width: 48px;
  max-width: 48px;
}
.rowId.active {
  //background: rgba(0, 135, 189, .1);
}
.rowId.active.selected {
  //background: #aaa
}
</style>

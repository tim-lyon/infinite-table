<template>
  <td :style="cellStyle">
      <div class="cell"
      :style="innerStyle"
      @mousedown="$emit('cellMouseDown')"
      @mouseover="$emit('cellMouseOver')">
        {{value}}
      </div>
  </td>
</template>

<script>

export default {
  name: 'InfiniteTableCell',
  props: [
    'config',
    'value',
    'selectedRange',
    'isActive',
    'rowIndex',
    'columnIndex'
  ],
  computed: {
    innerStyle(){
      return {
        borderTop: this.borderStyle(0),
        borderRight: this.borderStyle(1),
        borderBottom: this.borderStyle(2),
        borderLeft: this.borderStyle(3),
        padding: this.paddingWidths
      }
    },
    cellStyle(){
      let style = {
        background: this.isSelected && !this.isActive ? this.config.style.selection.fill.color : 'none',
        padding: 0
      }
      const color = this.fence.color
      let width = this.config.style.row.border.width
      let border = width + 'px solid ' + color
      if(this.isTop || this.isSelected){
        style.borderTop = border
      }
      if(this.isBottom || this.isSelected){
        style.borderBottom = border
      }
      width = this.config.style.column.border.width
      border = width + 'px solid ' + color
      if(this.isLeft || this.isSelected){
        style.borderLeft = border
      }
      if(this.isRight || this.isSelected){
        style.borderRight = border
      }
      return style
    },
    fence(){
      return this.config.style.selection.border
    },
    borderWidths(){
      return [
        this.isTop ? this.fence.width : 0,
        this.isRight ? this.fence.width : 0,
        this.isBottom ? this.fence.width : 0,
        this.isLeft ? this.fence.width : 0
      ]
    },
    paddingWidths(){
      return this.borderWidths.map(x => this.fence.width-x).join('px ') + 'px';
    },
    borderColors(){
      return [
        this.fence.color,
        this.isRight ? this.fence.color : this.config.style.column.border.color,
        this.isBottom ? this.fence.color: this.config.style.row.border.color,
        this.fence.color
      ]
    },
    isLeft(){
      return this.isInSelectedRowRange &&
      this.columnIndex === this.selectedRange.start.C;
    },
    isRight(){
      return this.isInSelectedRowRange &&
      this.columnIndex === this.selectedRange.end.C;
    },
    isTop(){
      return this.isInSelectedColumnRange &&
      this.rowIndex === this.selectedRange.start.R;
    },
    isBottom(){
      return this.isInSelectedColumnRange &&
      this.rowIndex === this.selectedRange.end.R;
    },
    isInSelectedRowRange(){
      return this.rowIndex >= this.selectedRange.start.R &&
      this.rowIndex <= this.selectedRange.end.R
    },
    isInSelectedColumnRange(){
      return this.columnIndex >= this.selectedRange.start.C &&
      this.columnIndex <= this.selectedRange.end.C
    },
    isSelected(){
      return this.isInSelectedRowRange && this.isInSelectedColumnRange
    }
  },
  methods: {
    borderStyle(i){
      return this.borderWidths[i] + 'px solid ' + this.borderColors[i]
    }
  }
}
</script>

<style>
.cell{
  width:6em;
  box-sizing: border-box;
}

::selection {
  background: none;
}
</style>

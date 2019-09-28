<template>
  <td :class="outerClass">
      <div :class="innerClass">
        <slot/>
      </div>
  </td>
</template>

<script>

export default {
  name: 'InfiniteTableCell',
  props: [
    'selectedRange',
    'activeCell',
    'rowIndex',
    'columnIndex'
  ],
  computed: {
    isActive(){
      return this.rowIndex == this.activeCell.R && this.columnIndex == this.activeCell.C
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
    innerClass(){
      return {
        cell: true,
        top: this.isTop,
        right: this.isRight,
        bottom: this.isBottom,
        left: this.isLeft,
        highlight: this.isSelected && !this.isActive
      }
    },
    outerClass(){
      return {
        outer: true,
        selected: this.isSelected
      }
    }
  }
}
</script>

<style>

.outer{
  padding:0;
}

.outer.selected{
  border:1px solid rgb(0, 135, 189);
}

.cell{
  width:6em;
  box-sizing: border-box;
  height: 30px;
  background:none;
  padding:1px;
  border:0px solid rgb(0, 135, 189);
}

.cell.top{
  border-top-width: 1px;
  padding-top: 0;
}

.cell.right{
  border-right-width: 1px;
  padding-right: 0;
}

.cell.bottom{
  border-bottom-width: 1px;
  padding-bottom: 0;
}

.cell.left{
  border-left-width: 1px;
  padding-left: 0;
}

.cell.highlight{
  background: rgba(0, 135, 189, .13);
}

::selection {
  background: none;
}
</style>

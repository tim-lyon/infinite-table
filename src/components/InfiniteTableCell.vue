<template>
  <td :class="cellClass" @mousedown="$emit('mousedown', $event)" @mouseover="$emit('mouseover', $event)" @dblclick="$emit('dblclick')">
      <slot/>
      <div :class="overlayClass" v-if="isTop || isBottom || isLeft || isRight"></div>
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
      return this.rowIndex == this.activeCell.R &&
      this.columnIndex == this.activeCell.C
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
      return this.isInSelectedRowRange &&
      this.isInSelectedColumnRange
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
    cellClass(){
      return {
        cell: true,
        highlight: this.isSelected,
        active: this.isActive
      }
    },
    overlayClass(){
      return{
        overlay: true,
        top: this.isTop,
        right: this.isRight,
        bottom: this.isBottom,
        left: this.isLeft,
      }
    }
  }
}
</script>

<style>

.cell{
  width: 6em;
  min-width: 6em;
  max-width: 6em;
  box-sizing: border-box;
  background:none;
  height:30px;
  overflow: visible;
  position:relative;
}

.cell.highlight{
  background: rgba(0, 135, 189, .1);
}
.cell.active{
  background: none;
}

.overlay {
  position: absolute;
  top: -1px;
  left: -1px;
  right: -1px;
  bottom: -1px;
  border: 0px solid rgb(0, 135, 189);
}
.overlay.top {
  border-top-width: 2px;
}
.overlay.left {
  border-left-width: 2px;
}
.overlay.right {
  border-right-width: 2px;
}
.overlay.bottom {
  border-bottom-width: 2px;
}
::selection {
  background: none;
}
</style>

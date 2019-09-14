<template>
  <div class="cell"
  @mousedown="$emit('cellMouseDown')"
  @mouseover="$emit('cellMouseOver')">
    <input class="cell" :value="value" @input="$emit('input', $event.target.value)"/>
  </div>
</template>

<script>

export default {
  name: 'InfiniteTableCell',
  props: ['value']
}
</script>

<style scoped>
.cell{
  width:6em;
  text-align: center;
  line-height:30px;
  height:30px;
  border: none;
  background: none;
  outline: none;
  cursor: cell;
  font-family:inherit;
  font-size: inherit;
}

::selection {
  background: none;
}
</style>

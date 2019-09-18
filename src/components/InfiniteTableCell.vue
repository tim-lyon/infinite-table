<template>
  <div class="cell"
  @mousedown="$emit('cellMouseDown')"
  @mouseover="$emit('cellMouseOver')">
    {{value}}
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
}

::selection {
  background: none;
}
</style>

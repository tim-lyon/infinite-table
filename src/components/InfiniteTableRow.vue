<template>
  <div class="row"
    @mousedown="$emit('rowMouseDown')"
    @mouseover="$emit('rowMouseOver')"
  >
    <InfiniteTableCell
      v-for="(cell, iCol) of data"
      :value="cell"
      @input="emitValue($event, iCol)"
      @cellMouseDown="$emit('cellMouseDown', iCol)"
      @cellMouseOver="$emit('cellMouseOver', iCol)"
      :key="iCol"
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
  props: {
    data: {
      type: Array,
      required: true
    }
  },
  created(){
    this.$emit('created')
  },
  methods: {
    emitValue(value, column) {
      this.$emit('input', {value,column})
    }
  }
}
</script>

<style scoped>
.row{
  display:flex;
  line-height:30px;
}
</style>

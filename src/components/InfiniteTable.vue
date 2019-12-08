<template>
  <div class="root-element">
    <InfiniteTableWrapper
      v-if="useWrapper"
      v-model="value"
      :headers="headers"
      :editable="editable"
      :selectable="selectable"
    />
    <InfiniteTableCore
      v-else
      :headers="headers"
      :row-count="rowCount"
      :column-count="columnCount"
      :get="get"
      :editable="editable"
      :selectable="selectable"
      @set="onSet"
      @cut="onCut"
      @copy="onCopy"
      @paste="onPaste"
      @undo="$emit('undo')"
      @redo="$emit('redo')"
      @buttonClick="onButtonClick"
    />
  </div>
</template>

<script>
import InfiniteTableCore from "./InfiniteTableCore.vue";
import InfiniteTableWrapper from "./InfiniteTableWrapper.vue";

export default {
  name: "InfiniteTable",
  props: {
    value: {
      type: Array,
      required: false,
      default: null
    },
    columnCount: {
      type: Number,
      required: false
    },
    rowCount: {
      type: Number,
      required: false
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
    get: {
      type: Function,
      required: false
    }
  },
  components: {
    InfiniteTableCore,
    InfiniteTableWrapper
  },
  computed: {
    useWrapper() {
      return this.value != null;
    }
  },
  methods: {
    onSet(payload) {
      this.$emit("set", payload);
    },
    onCut(payload) {
      this.$emit("cut", payload);
    },
    onCopy(payload) {
      this.$emit("copy", payload);
    },
    onPaste(payload) {
      this.$emit("paste", payload);
    },
    onButtonClick(payload) {
      this.$emit("buttonClick", payload);
    }
  }
};
</script>

<style scoped>
.root-element {
  max-height: 400px;
}
</style>
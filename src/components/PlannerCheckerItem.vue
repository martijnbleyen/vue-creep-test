<template>
  <div class="planner-checker-item">
    <span v-for="i in 9" :key="i"> {{ item }}.{{ i }} </span>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, onUnmounted } from "vue";
import { eventBus } from "../eventBus";

export default defineComponent({
  components: {},
  props: {
    item: Number,
  },
  setup() {
    onMounted(() => {
      eventBus.$emit("TRIGGER_PLANNER_CHECKER");
    });
    onUnmounted(() => {
      eventBus.$emit("TRIGGER_PLANNER_CHECKER");
    });
    return {};
  },
});
</script>
<style lang="scss">
.planner-checker-item {
  font-size: 3rem;
  font-weight: bold;
  height: 100px;
  background: rgba(#ccc, 0.2);
  overflow: hidden;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-column-gap: 1px;
  grid-row-gap: 1px;
  font-size: 12px;
  > span {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>

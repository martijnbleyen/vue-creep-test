<template>
  <div>
    <button @click="startPerf">Start</button>
    <div class="stats-container">
      <div class="stats-block">
        <div class="title">On Mount</div>
        <div class="stats">
          <div>
            <span>{{ prevResults.mount.length - 1 }}</span>
            <span>Total Mounts</span>
          </div>
          <div>
            <span>{{ stats.mount.min.toFixed(2) }}ms</span>
            <span>Min</span>
          </div>
          <div>
            <span>{{ stats.mount.max.toFixed(2) }}ms</span>
            <span>Max</span>
          </div>
          <div>
            <span>{{ stats.mount.avg.toFixed(2) }}ms</span>
            <span>Avg</span>
          </div>
          <div>
            <span
              :class="{
                'success-text': stats.mount.diffLast <= 10,
                'warning-text': stats.mount.diffLast > 10,
                'error-text': stats.mount.diffLast > 50,
              }"
              >{{ stats.mount.diffLast.toFixed(2) }}%</span
            >
            <span>Diff last run</span>
          </div>
          <div>
            <span
              :class="{
                'success-text': stats.mount.diffFirst <= 10,
                'warning-text': stats.mount.diffFirst > 10,
                'error-text': stats.mount.diffFirst > 50,
              }"
              >{{ stats.mount.diffFirst.toFixed(2) }}%</span
            >
            <span>Diff first run</span>
          </div>
        </div>
      </div>
      <div class="stats-block">
        <div class="title">On UnMount</div>
        <div class="stats">
          <div>
            <span>{{ prevResults.unmount.length - 1 }}</span>
            <span>Total Unmounts</span>
          </div>
          <div>
            <span>{{ stats.unmount.min.toFixed(2) }}ms</span>
            <span>Min</span>
          </div>
          <div>
            <span>{{ stats.unmount.max.toFixed(2) }}ms</span>
            <span>Max</span>
          </div>
          <div>
            <span>{{ stats.unmount.avg.toFixed(2) }}ms</span>
            <span>Avg</span>
          </div>
          <div>
            <span
              :class="{
                'success-text': stats.unmount.diffLast <= 10,
                'warning-text': stats.unmount.diffLast > 10,
                'error-text': stats.unmount.diffLast > 50,
              }"
              >{{ stats.unmount.diffLast.toFixed(2) }}%</span
            >
            <span>Diff last run</span>
          </div>
          <div>
            <span
              :class="{
                'success-text': stats.unmount.diffFirst <= 10,
                'warning-text': stats.unmount.diffFirst > 10,
                'error-text': stats.unmount.diffFirst > 50,
              }"
              >{{ stats.unmount.diffFirst.toFixed(2) }}%</span
            >
            <span>Diff first run</span>
          </div>
        </div>
      </div>
    </div>
    <div class="planner-checker">
      <planner-checker-item v-for="item in allRows" :key="item" :item="item" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from "vue";
import { eventBus } from "../eventBus";
import PlannerCheckerItem from "./PlannerCheckerItem.vue";

export default defineComponent({
  components: {
    PlannerCheckerItem,
  },
  setup() {
    const allRows = ref<number[]>([]);

    const performanceTimer = ref(0);

    const prevResults = ref<{
      mount: number[];
      unmount: number[];
      mountAvg: number;
      unmountAvg: number;
    }>({
      mount: [0],
      unmount: [0],
      mountAvg: 0,
      unmountAvg: 0,
    });

    const toggleCount = ref(0);
    const totalToggles = ref(0);

    const firstMountAvg = ref(0);
    const firstUnmountAvg = ref(0);

    const lastMountAvg = ref(0);
    const lastUnmountAvg = ref(0);

    const MAX_TOGGLE_COUNT = 200;

    function percDiff(original: number, newNumber: number) {
      if (!original || !newNumber) {
        return 0;
      }
      const increase = newNumber - original;
      const percentageIncrease = (increase / original) * 100;
      return percentageIncrease;
    }

    const stats = computed(() => {
      console.log("getting stats");
      const mountAvg =
        prevResults.value.mount.length <= 1
          ? 0
          : prevResults.value.mount.reduce((a, b) => a + b, 0) /
            (prevResults.value.mount.length - 1);
      const unmountAvg =
        prevResults.value.unmount.length <= 1
          ? 0
          : prevResults.value.unmount.reduce((a, b) => a + b, 0) /
            (prevResults.value.unmount.length - 1);

      const mountLen = prevResults.value.mount.length;
      const unmountLen = prevResults.value.unmount.length;

      return {
        mount: {
          min: mountLen > 1 ? Math.min(...prevResults.value.mount.slice(1)) : 0,
          max: Math.max(...prevResults.value.mount),
          avg: mountAvg,
          diffLast: percDiff(lastMountAvg.value, mountAvg),
          diffFirst: percDiff(firstMountAvg.value, mountAvg),
        },
        unmount: {
          min:
            unmountLen > 1
              ? Math.min(...prevResults.value.unmount.slice(1))
              : 0,
          max: Math.max(...prevResults.value.unmount),
          avg: unmountAvg,
          diffLast: percDiff(lastUnmountAvg.value, unmountAvg),
          diffFirst: percDiff(firstUnmountAvg.value, unmountAvg),
        },
      };
    });

    function startPerf() {
      toggleCount.value++;
      totalToggles.value++;

      if (toggleCount.value === 1) {
        lastMountAvg.value = stats.value.mount.avg;
        lastUnmountAvg.value = stats.value.unmount.avg;
      }

      if (toggleCount.value > MAX_TOGGLE_COUNT) {
        toggleCount.value = 0;
        if (!firstMountAvg.value) {
          firstMountAvg.value = stats.value.mount.avg;
          firstUnmountAvg.value = stats.value.unmount.avg;
        }
        console.log(prevResults.value);
        return;
      }

      const hasRows = allRows.value.length;
      eventBus.$once("TRIGGER_PLANNER_CHECKER", () => {
        const now = new Date().getTime();
        const diff = now - performanceTimer.value;

        const hasRows = allRows.value.length;

        if (hasRows) {
          prevResults.value.mount.push(diff);
        } else {
          prevResults.value.unmount.push(diff);
        }
        setTimeout(startPerf, 10);
      });

      performanceTimer.value = new Date().getTime();

      allRows.value = hasRows
        ? []
        : new Array(100).fill(0).map((_, index) => index);
    }

    return {
      startPerf,
      allRows,
      stats,
      prevResults,
    };
  },
});
</script>

<style lang="scss">
.planner-checker {
  height: 500px;
  border: 2px solid #5b6078;
  overflow: auto;
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-column-gap: 1rem;
  grid-row-gap: 1rem;
}

.stats-container {
  font-family: "Fira Code", "Monaco", monospace;
  display: flex;
  gap: 2rem;
  margin-bottom: 2rem;
  margin-top: 2rem;
  .stats-block {
    color: #bdc5e6;
    background: #24273a;
    border: 1px solid #5b6078;
    padding: 1rem;
    flex: 1;
    border-radius: 5px;
    .title {
      font-size: 1.5rem;
    }
    .stats {
      font-size: 1.25rem;
      margin-top: 0.5rem;
      > div {
        display: flex;
        margin-top: 0.5rem;
        gap: 2rem;
        span {
          flex: 1;
          &:first-child {
            flex: 0 0 150px;
            text-align: right;
            color: #c6a0f6;
          }
          &.success-text {
            color: #8bd5ca;
          }
          &.warning-text {
            color: #df9b78;
          }
          &.error-text {
            color: #e98594;
          }
        }
      }
    }
  }
}
</style>

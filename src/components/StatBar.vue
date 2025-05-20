<template>
  <div class="d-flex align-items-center gap-3 mb-2">
    <span class="text-capitalize flex-shrink-0" style="width: 100px;">{{ stat.stat.name }}</span>
    <div class="progress flex-grow-1" style="height: 20px;">
      <div
        class="progress-bar bg-warning"
        role="progressbar"
        :style="{ width: barWidth + '%', transition: 'width 0.5s ease' }"
        :aria-valuenow="stat.base_stat"
        aria-valuemin="0"
        aria-valuemax="150"
      >
        {{ stat.base_stat }}
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  stat: Object,
});

const maxStat = 150;
const barWidth = Math.min((props.stat.base_stat / maxStat) * 100, 100);
</script>

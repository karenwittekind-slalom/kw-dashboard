<script setup lang="ts">
const props = defineProps<{
  label: string
  icon: string
  value: string
  direction: number
  accent: string
  helper: string
}>()
</script>

<template>
  <v-card class="stat-card" rounded="xl" elevation="0" color="surfaceVariant" border>
    <v-card-text class="pa-5">
      <div class="d-flex justify-space-between align-center mb-3">
        <div class="dashboard-card-title metric-title">{{ props.label }}</div>
        <v-avatar :color="`${props.accent}-lighten-3`" size="36" class="icon-badge">
          <v-icon :icon="props.icon" size="18" />
        </v-avatar>
      </div>

      <div class="text-h5 font-weight-bold mb-1">{{ props.value }}</div>
      <div class="d-flex align-center gap-2">
        <span class="trend-pill" :class="props.direction >= 0 ? 'positive' : 'negative'">
          <v-icon size="14">
            {{ props.direction >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
          </v-icon>
          {{ props.direction >= 0 ? '+' : '' }}{{ props.direction.toFixed(1) }}%
        </span>
        <span class="dashboard-card-caption text-medium-emphasis">{{ props.helper }}</span>
      </div>
    </v-card-text>
  </v-card>
</template>

<style scoped>
.stat-card {
  height: 100%;
}

:global(.v-theme--light .metric-title) {
  color: rgb(var(--v-theme-on-surface)) !important;
}

.icon-badge {
  opacity: 0.9;
}

.trend-pill {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  border-radius: 999px;
  padding: 4px 8px;
  font-size: 0.73rem;
  font-weight: 600;
}

.trend-pill.positive {
  background: rgba(52, 211, 153, 0.12);
  color: #86efac;
}

.trend-pill.negative {
  background: rgba(248, 113, 113, 0.12);
  color: #fca5a5;
}

:global(.v-theme--light .trend-pill.positive) {
  background: rgba(var(--v-theme-success), 0.12);
  color: rgb(var(--v-theme-success));
}

:global(.v-theme--light .trend-pill.negative) {
  background: rgba(var(--v-theme-on-surface-variant), 0.08);
  color: rgb(var(--v-theme-on-surface-variant));
}
</style>
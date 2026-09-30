<script setup lang="ts">
import { computed, ref } from 'vue'
import { Bar, Line } from 'vue-chartjs'
import MetricCard from '@/components/MetricCard.vue'
import {
  BarElement,
  CategoryScale,
  Chart,
  Filler,
  Legend,
  LineElement,
  LinearScale,
  PointElement,
  Title,
  Tooltip,
} from 'chart.js'
import { useTheme } from 'vuetify'
import metricsData from '@/data/metrics.json'

Chart.register(CategoryScale, LinearScale, BarElement, PointElement, LineElement, Filler, Title, Tooltip, Legend)

type Metric = {
  month: string
  date: string
  revenue: number
  visitors: number
  conversions: number
  orders: number
}

const metrics: Metric[] = metricsData as Metric[]
const selectedMonth = ref('All')
const monthOptions = [
  { title: 'All months', value: 'All' },
  ...metrics.map((metric) => ({ title: metric.month, value: metric.month })),
]

const theme = useTheme()
const isDark = computed(() => theme.global.current.value.dark)

const toggleTheme = () => {
  theme.global.name.value = isDark.value ? 'light' : 'dark'
}

const getMonthMetric = (month: string): Metric =>
  metrics.find((entry) => entry.month === month) ?? metrics[0]!

const getPreviousMetric = (month: string): Metric => {
  if (month === 'All') {
    return metrics[metrics.length - 2] ?? metrics[metrics.length - 1] ?? metrics[0]!
  }

  const index = metrics.findIndex((entry) => entry.month === month)
  return metrics[index - 1] ?? metrics[index] ?? metrics[0]!
}

const formatCurrency = (value: number) =>
  new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    maximumFractionDigits: 0,
  }).format(value)

const formatNumber = (value: number) => new Intl.NumberFormat('en-US').format(value)

const formatPercent = (value: number) => `${value.toFixed(1)}%`

const percentChange = (current: number, previous: number) => {
  if (!previous) return 0
  return ((current - previous) / previous) * 100
}

const totals = computed(() => ({
  revenue: metrics.reduce((sum, item) => sum + item.revenue, 0),
  visitors: metrics.reduce((sum, item) => sum + item.visitors, 0),
  conversions: metrics.reduce((sum, item) => sum + item.conversions, 0) / metrics.length,
  orders: metrics.reduce((sum, item) => sum + item.orders, 0),
}))

const activeMetric = computed(() => {
  if (selectedMonth.value === 'All') {
    return {
      revenue: totals.value.revenue,
      visitors: totals.value.visitors,
      conversions: totals.value.conversions,
      orders: totals.value.orders,
    }
  }

  const monthMetric = getMonthMetric(selectedMonth.value)
  return {
    revenue: monthMetric.revenue,
    visitors: monthMetric.visitors,
    conversions: monthMetric.conversions,
    orders: monthMetric.orders,
  }
})

const previousMetric = computed(() => {
  if (selectedMonth.value === 'All') {
    const previous = getPreviousMetric('All')
    return {
      revenue: previous.revenue,
      visitors: previous.visitors,
      conversions: previous.conversions,
      orders: previous.orders,
    }
  }

  const previous = getPreviousMetric(selectedMonth.value)
  return {
    revenue: previous.revenue,
    visitors: previous.visitors,
    conversions: previous.conversions,
    orders: previous.orders,
  }
})

const statCards = computed(() => [
  {
    label: 'Revenue',
    value: selectedMonth.value === 'All' ? formatCurrency(totals.value.revenue) : formatCurrency(activeMetric.value.revenue),
    direction: percentChange(activeMetric.value.revenue, previousMetric.value.revenue),
    accent: 'primary',
    helper: selectedMonth.value === 'All' ? 'annual total' : 'month total',
    icon: 'mdi-currency-usd',
  },
  {
    label: 'Visitors',
    value: selectedMonth.value === 'All' ? formatNumber(totals.value.visitors) : formatNumber(activeMetric.value.visitors),
    direction: percentChange(activeMetric.value.visitors, previousMetric.value.visitors),
    accent: 'info',
    helper: selectedMonth.value === 'All' ? 'annual total' : 'month total',
    icon: 'mdi-account-group',
  },
  {
    label: 'Conversions',
    value: selectedMonth.value === 'All' ? formatPercent(totals.value.conversions) : formatPercent(activeMetric.value.conversions),
    direction: percentChange(activeMetric.value.conversions, previousMetric.value.conversions),
    accent: 'success',
    helper: selectedMonth.value === 'All' ? 'average rate' : 'monthly rate',
    icon: 'mdi-chart-line',
  },
  {
    label: 'Orders',
    value: selectedMonth.value === 'All' ? formatNumber(totals.value.orders) : formatNumber(activeMetric.value.orders),
    direction: percentChange(activeMetric.value.orders, previousMetric.value.orders),
    accent: 'warning',
    helper: selectedMonth.value === 'All' ? 'annual total' : 'month total',
    icon: 'mdi-cart-outline',
  },
])

const selectedChartIndex = computed(() =>
  selectedMonth.value === 'All' ? -1 : metrics.findIndex((item) => item.month === selectedMonth.value),
)

const chartView = computed(() => ({
  labels: metrics.map((item) => item.month),
  revenue: metrics.map((item) => item.revenue),
  visitors: metrics.map((item) => item.visitors),
  conversions: metrics.map((item) => item.conversions),
}))

const revenueChartData = computed(() => ({
  labels: chartView.value.labels,
  datasets: [
    {
      label: 'Revenue',
      data: chartView.value.revenue,
      backgroundColor: chartView.value.revenue.map((_, index) =>
        selectedChartIndex.value === -1 || selectedChartIndex.value === index
          ? isDark.value ? 'rgba(129, 140, 248, 0.9)' : 'rgba(79, 70, 229, 0.9)'
          : isDark.value ? 'rgba(129, 140, 248, 0.28)' : 'rgba(79, 70, 229, 0.2)',
      ),
      borderRadius: 8,
      borderSkipped: false,
      maxBarThickness: 42,
    },
  ],
}))

const visitorsChartData = computed(() => ({
  labels: chartView.value.labels,
  datasets: [
    {
      label: 'Visitors',
      data: chartView.value.visitors,
      borderColor: isDark.value ? '#60a5fa' : '#2563eb',
      backgroundColor: isDark.value ? 'rgba(96, 165, 250, 0.18)' : 'rgba(37, 99, 235, 0.12)',
      pointBackgroundColor: chartView.value.visitors.map((_, index) =>
        selectedChartIndex.value === -1 || selectedChartIndex.value === index
          ? isDark.value ? '#7dd3fc' : '#2563eb'
          : isDark.value ? 'rgba(96, 165, 250, 0.35)' : 'rgba(37, 99, 235, 0.45)',
      ),
      pointBorderColor: isDark.value ? '#e0f2fe' : '#ffffff',
      pointRadius: chartView.value.visitors.map((_, index) =>
        selectedChartIndex.value === index ? 7 : 3,
      ),
      pointHoverRadius: chartView.value.visitors.map((_, index) =>
        selectedChartIndex.value === index ? 9 : 6,
      ),
      fill: true,
      tension: 0.35,
    },
  ],
}))

const conversionChartData = computed(() => ({
  labels: chartView.value.labels,
  datasets: [
    {
      label: 'Conversion Rate',
      data: chartView.value.conversions,
      borderColor: isDark.value ? '#34d399' : '#059669',
      backgroundColor: isDark.value ? 'rgba(52, 211, 153, 0.16)' : 'rgba(5, 150, 105, 0.13)',
      pointBackgroundColor: chartView.value.conversions.map((_, index) =>
        selectedChartIndex.value === -1 || selectedChartIndex.value === index
          ? isDark.value ? '#a7f3d0' : '#059669'
          : isDark.value ? 'rgba(52, 211, 153, 0.35)' : 'rgba(5, 150, 105, 0.45)',
      ),
      pointBorderColor: isDark.value ? '#ecfdf5' : '#ffffff',
      pointRadius: chartView.value.conversions.map((_, index) =>
        selectedChartIndex.value === index ? 7 : 3,
      ),
      pointHoverRadius: chartView.value.conversions.map((_, index) =>
        selectedChartIndex.value === index ? 9 : 6,
      ),
      fill: true,
      tension: 0.4,
    },
  ],
}))

const selectedValueLabelPlugin = {
  id: 'selectedValueLabel',
  afterDatasetsDraw(chart: Chart) {
    const index = chart.data.labels?.indexOf(selectedMonth.value) ?? -1
    const dataset = chart.data.datasets[0]
    const current = Number(dataset?.data[index])
    const previous = index > 0 ? Number(dataset?.data[index - 1]) : undefined

    if (index < 0 || !dataset || !Number.isFinite(current)) return

    let valueLabel = ''
    let changeValue = ''
    if (dataset.label === 'Revenue') {
      valueLabel = formatCurrency(current)
      if (previous !== undefined && Number.isFinite(previous)) {
        changeValue = formatCurrency(Math.abs(current - previous))
      }
    } else if (dataset.label === 'Visitors') {
      valueLabel = formatNumber(current)
      if (previous !== undefined && Number.isFinite(previous)) {
        changeValue = formatNumber(Math.abs(current - previous))
      }
    } else {
      valueLabel = formatPercent(current)
      if (previous !== undefined && Number.isFinite(previous)) {
        changeValue = `${Math.abs(current - previous).toFixed(1)} pp`
      }
    }

    const delta = previous === undefined ? undefined : current - previous
    const changeLabel = delta === undefined
      ? ''
      : `${delta >= 0 ? '\u2191' : '\u2193'} ${changeValue}`
    const element = chart.getDatasetMeta(0).data[index]
    if (!element) return

    const { x, y } = element.getProps(['x', 'y'], true)
    const { ctx, chartArea } = chart
    const height = changeLabel ? 42 : 28
    const padding = 10

    ctx.save()
    ctx.font = '600 12px sans-serif'
    const valueWidth = ctx.measureText(valueLabel).width
    ctx.font = '500 10px sans-serif'
    const changeWidth = changeLabel ? ctx.measureText(changeLabel).width : 0
    const width = Math.min(Math.max(valueWidth, changeWidth) + padding * 2, chartArea.width - 4)
    const left = Math.max(
      chartArea.left + 2,
      Math.min(x - width / 2, chartArea.right - width - 2),
    )
    let top = y - height - 10
    if (top < chartArea.top + 2) top = y + 12
    top = Math.max(chartArea.top + 2, Math.min(top, chartArea.bottom - height - 2))

    ctx.beginPath()
    ctx.roundRect(left, top, width, height, 6)
    ctx.fillStyle = isDark.value ? 'rgba(15, 23, 42, 0.92)' : 'rgba(255, 255, 255, 0.96)'
    ctx.fill()
    ctx.strokeStyle = isDark.value ? 'rgba(148, 163, 184, 0.3)' : 'rgba(71, 85, 105, 0.25)'
    ctx.lineWidth = 1
    ctx.stroke()

    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.font = '600 12px sans-serif'
    ctx.fillStyle = isDark.value ? '#e2e8f0' : '#0f172a'
    ctx.fillText(valueLabel, left + width / 2, top + (changeLabel ? 13 : 14), width - padding)

    if (changeLabel) {
      ctx.font = '500 10px sans-serif'
      ctx.fillStyle = delta! >= 0
        ? isDark.value
          ? '#86efac'
          : `rgb(${getComputedStyle(chart.canvas).getPropertyValue('--v-theme-success')})`
        : isDark.value
          ? '#fca5a5'
          : `rgb(${getComputedStyle(chart.canvas).getPropertyValue('--v-theme-error')})`
      ctx.fillText(changeLabel, left + width / 2, top + 30, width - padding)
    }
    ctx.restore()
  },
}

Chart.register(selectedValueLabelPlugin)

const chartOptions = computed(() => ({
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      display: false,
    },
    tooltip: {
      backgroundColor: isDark.value ? 'rgba(15, 23, 42, 0.9)' : 'rgba(255, 255, 255, 0.98)',
      titleColor: isDark.value ? '#f8fafc' : '#0f172a',
      bodyColor: isDark.value ? '#e2e8f0' : '#334155',
      borderColor: isDark.value ? 'rgba(148, 163, 184, 0.3)' : 'rgba(71, 85, 105, 0.25)',
      borderWidth: 1,
      displayColors: false,
      callbacks: {
        label: (context: any) => {
          const label = context.dataset.label ?? ''
          const value = context.parsed.y ?? 0

          if (label === 'Revenue') return `${label}: ${formatCurrency(value)}`
          if (label === 'Visitors') return `${label}: ${formatNumber(value)}`
          return `${label}: ${formatPercent(value)}`
        },
      },
    },
  },
  scales: {
    x: {
      grid: {
        display: false,
      },
      ticks: {
        color: isDark.value ? '#94a3b8' : '#64748b',
      },
    },
    y: {
      grid: {
        color: isDark.value ? 'rgba(148, 163, 184, 0.15)' : 'rgba(100, 116, 139, 0.18)',
      },
      ticks: {
        color: isDark.value ? '#94a3b8' : '#64748b',
      },
    },
  },
}))
</script>

<template>
  <v-app>
    <v-app-bar flat class="px-4 dashboard-app-bar" color="surface" border>
      <div class="d-flex align-center">
        <span class="text-h6 font-weight-bold dashboard-title">Analytics Dashboard</span>
      </div>

      <v-spacer />

      <v-select
        v-model="selectedMonth"
        :items="monthOptions"
        item-title="title"
        item-value="value"
        variant="outlined"
        density="comfortable"
        hide-details
        class="month-picker"
        prepend-inner-icon="mdi-calendar"
      />

      <v-btn
        icon
        variant="text"
        class="ml-3 theme-toggle"
        @click="toggleTheme"
        :aria-label="isDark ? 'Switch to light theme' : 'Switch to dark theme'"
      >
        <v-icon :icon="isDark ? 'mdi-weather-night' : 'mdi-weather-sunny'" size="18" />
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container fluid class="dashboard-shell">
        <v-row gap="12">
          <v-col v-for="card in statCards" :key="card.label" cols="12" sm="6" md="3">
            <MetricCard
              :label="card.label"
              :icon="card.icon"
              :value="card.value"
              :direction="card.direction"
              :accent="card.accent"
              :helper="card.helper"
            />
          </v-col>
        </v-row>

        <v-row gap="12">
          <v-col cols="12" lg="6">
            <v-card class="chart-card" rounded="xl" elevation="0" color="surfaceVariant" border>
              <v-card-title class="chart-card-title d-flex justify-space-between align-center pt-5 pb-2">
                <span class="dashboard-card-title">Monthly Revenue</span>
                <span class="dashboard-card-caption text-medium-emphasis">
                  {{ selectedMonth === 'All' ? 'Full year' : selectedMonth }}
                </span>
              </v-card-title>
              <v-card-text class="chart-panel">
                <Bar :data="revenueChartData" :options="chartOptions" />
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12" lg="6">
            <v-card class="chart-card" rounded="xl" elevation="0" color="surfaceVariant" border>
              <v-card-title class="chart-card-title d-flex justify-space-between align-center pt-5 pb-2">
                <span class="dashboard-card-title">Visitors</span>
                <span class="dashboard-card-caption text-medium-emphasis">
                  {{ selectedMonth === 'All' ? '12-month trend' : selectedMonth }}
                </span>
              </v-card-title>
              <v-card-text class="chart-panel">
                <Line :data="visitorsChartData" :options="chartOptions" />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <v-row gap="12">
          <v-col cols="12">
            <v-card class="chart-card" rounded="xl" elevation="0" color="surfaceVariant" border>
              <v-card-title class="chart-card-title d-flex justify-space-between align-center pt-5 pb-2">
                <span class="dashboard-card-title">Conversion Rate</span>
                <span class="dashboard-card-caption text-medium-emphasis">
                  {{ selectedMonth === 'All' ? 'Full year' : selectedMonth }}
                </span>
              </v-card-title>
              <v-card-text class="chart-panel large-panel">
                <Line :data="conversionChartData" :options="chartOptions" />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<style scoped>
.dashboard-shell {
  max-width: 1440px;
  padding: 24px 24px 40px;
}

.month-picker {
  max-width: 180px;
}

.month-picker :deep(.v-select__selection-text) {
  font-size: 0.8125rem;
}

.month-picker :deep(.v-icon) {
  width: 16px !important;
  height: 16px !important;
  font-size: 16px !important;
}

:global(.v-theme--light .dashboard-title),
:global(.v-theme--light .chart-card-title) {
  color: rgb(var(--v-theme-on-surface)) !important;
}

:global(.v-theme--light .month-picker .v-field__input),
:global(.v-theme--light .month-picker .v-select__selection-text),
:global(.v-theme--light .month-picker .v-icon),
:global(.v-theme--light .theme-toggle) {
  color: rgb(var(--v-theme-on-surface)) !important;
  opacity: 1;
}

:global(.v-theme--light .month-picker .v-field__outline) {
  color: rgb(var(--v-theme-on-surface-variant)) !important;
}

.chart-panel {
  height: 260px;
  padding-top: 8px;
}

.large-panel {
  height: 300px;
}

@media (max-width: 600px) {
  .dashboard-shell {
    padding-left: 12px;
    padding-right: 12px;
  }

  .month-picker {
    max-width: 140px;
  }
}
</style>

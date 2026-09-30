<script setup lang="ts">
import { computed, ref } from 'vue'
import { Bar, Line } from 'vue-chartjs'
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

const getTrendLabel = (value: number) => `${value >= 0 ? '+' : ''}${value.toFixed(1)}%`

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
          ? 'rgba(129, 140, 248, 0.9)'
          : 'rgba(129, 140, 248, 0.28)',
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
      borderColor: '#60a5fa',
      backgroundColor: 'rgba(96, 165, 250, 0.18)',
      pointBackgroundColor: chartView.value.visitors.map((_, index) =>
        selectedChartIndex.value === -1 || selectedChartIndex.value === index
          ? '#7dd3fc'
          : 'rgba(96, 165, 250, 0.35)',
      ),
      pointBorderColor: '#e0f2fe',
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
      borderColor: '#34d399',
      backgroundColor: 'rgba(52, 211, 153, 0.16)',
      pointBackgroundColor: chartView.value.conversions.map((_, index) =>
        selectedChartIndex.value === -1 || selectedChartIndex.value === index
          ? '#a7f3d0'
          : 'rgba(52, 211, 153, 0.35)',
      ),
      pointBorderColor: '#ecfdf5',
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
    ctx.fillStyle = 'rgba(15, 23, 42, 0.92)'
    ctx.fill()
    ctx.strokeStyle = 'rgba(148, 163, 184, 0.3)'
    ctx.lineWidth = 1
    ctx.stroke()

    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.font = '600 12px sans-serif'
    ctx.fillStyle = '#e2e8f0'
    ctx.fillText(valueLabel, left + width / 2, top + (changeLabel ? 13 : 14), width - padding)

    if (changeLabel) {
      ctx.font = '500 10px sans-serif'
      ctx.fillStyle = delta! >= 0 ? '#86efac' : '#fca5a5'
      ctx.fillText(changeLabel, left + width / 2, top + 30, width - padding)
    }
    ctx.restore()
  },
}

Chart.register(selectedValueLabelPlugin)

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      display: false,
    },
    tooltip: {
      backgroundColor: 'rgba(15, 23, 42, 0.9)',
      titleColor: '#f8fafc',
      bodyColor: '#e2e8f0',
      borderColor: 'rgba(148, 163, 184, 0.3)',
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
        color: '#94a3b8',
      },
    },
    y: {
      grid: {
        color: 'rgba(148, 163, 184, 0.15)',
      },
      ticks: {
        color: '#94a3b8',
      },
    },
  },
}
</script>

<template>
  <v-app>
    <v-app-bar flat class="px-4" color="surface" border>
      <div class="d-flex align-center">
        <span class="text-h6 font-weight-bold">Analytics Dashboard</span>
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
        class="ml-3"
        @click="toggleTheme"
        :aria-label="isDark ? 'Switch to light theme' : 'Switch to dark theme'"
      >
        <v-icon>{{ isDark ? 'mdi-weather-sunny' : 'mdi-weather-night' }}</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container fluid class="dashboard-shell">
        <v-row class="mb-4" density="compact">
          <v-col v-for="card in statCards" :key="card.label" cols="12" sm="6" md="3">
            <v-card class="stat-card" rounded="xl" elevation="0" color="surfaceVariant" border>
              <v-card-text class="pa-5">
                <div class="d-flex justify-space-between align-center mb-3">
                  <div class="text-body-2 text-medium-emphasis">{{ card.label }}</div>
                  <v-avatar :color="card.accent + '-lighten-3'" size="36" class="icon-badge">
                    <v-icon :icon="card.icon" size="18" />
                  </v-avatar>
                </div>

                <div class="text-h5 font-weight-bold mb-1">{{ card.value }}</div>
                <div class="d-flex align-center gap-2">
                  <span
                    class="trend-pill"
                    :class="card.direction >= 0 ? 'positive' : 'negative'"
                  >
                    <v-icon size="14">{{ card.direction >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down' }}</v-icon>
                    {{ getTrendLabel(card.direction) }}
                  </span>
                  <span class="text-caption text-medium-emphasis">{{ card.helper }}</span>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <v-row class="mb-4" density="compact">
          <v-col cols="12" lg="6">
            <v-card class="chart-card" rounded="xl" elevation="0" color="surfaceVariant" border>
              <v-card-title class="d-flex justify-space-between align-center pb-2">
                <span>Monthly Revenue</span>
                <span class="text-caption text-medium-emphasis">
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
              <v-card-title class="d-flex justify-space-between align-center pb-2">
                <span>Visitors</span>
                <span class="text-caption text-medium-emphasis">
                  {{ selectedMonth === 'All' ? '12-month trend' : selectedMonth }}
                </span>
              </v-card-title>
              <v-card-text class="chart-panel">
                <Line :data="visitorsChartData" :options="chartOptions" />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>

        <v-row density="compact">
          <v-col cols="12">
            <v-card class="chart-card" rounded="xl" elevation="0" color="surfaceVariant" border>
              <v-card-title class="d-flex justify-space-between align-center pb-2">
                <span>Conversion Rate</span>
                <span class="text-caption text-medium-emphasis">
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

.stat-card {
  height: 100%;
  background: rgba(15, 23, 42, 0.7);
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

.chart-card {
  background: rgba(15, 23, 42, 0.68);
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

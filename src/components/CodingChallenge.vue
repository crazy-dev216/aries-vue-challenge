<template>
  <div>
    <h1>Options Profit Calculator</h1>
    <div>
      <line-chart :chartData="chartData" :options="chartOptions"></line-chart>
    </div>
    <div v-if="calculation">
      <h2>Max Profit: {{ calculation.maxProfit }}</h2>
      <h2>Max Loss: {{ calculation.maxLoss }}</h2>
      <h2>Break Even Points: {{ calculation.breakEvenPoints.join(', ') }}</h2>
    </div>
  </div>
</template>

<script>
import { Line } from 'vue-chartjs'
import { defineComponent } from 'vue'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

export default defineComponent({
  name: 'CodingChallenge',
  components: {
    LineChart: Line
  },
  props: {
    optionsData: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      chartData: {
        labels: [],
        datasets: [
          {
            label: 'Profit/Loss',
            backgroundColor: '#f87979',
            data: []
          }
        ]
      },
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false
      },
      calculation: null
    }
  },
  watch: {
    optionsData: {
      handler(newData) {
        this.calculateRiskReward(newData)
      },
      immediate: true
    }
  },
  methods: {
    calculateRiskReward(options) {
      // Initialize variables
      const step = 1 // Step for underlying price range
      const minPrice = Math.min(...options.map(o => o.strike_price)) - 10
      const maxPrice = Math.max(...options.map(o => o.strike_price)) + 10
      const prices = []
      const profits = []

      let maxProfit = -Infinity
      let maxLoss = Infinity
      let breakEvenPoints = []

      // Calculate profit/loss for each price point
      for (let price = minPrice; price <= maxPrice; price += step) {
        let totalProfit = 0

        options.forEach(option => {
          const { strike_price, type, long_short, ask, bid } = option
          const premium = (ask + bid) / 2
          const quantity = long_short === 'long' ? 1 : -1

          if (type === 'Call') {
            totalProfit += quantity * (Math.max(price - strike_price, 0) - premium)
          } else if (type === 'Put') {
            totalProfit += quantity * (Math.max(strike_price - price, 0) - premium)
          }
        })

        prices.push(price)
        profits.push(totalProfit)

        if (totalProfit > maxProfit) maxProfit = totalProfit
        if (totalProfit < maxLoss) maxLoss = totalProfit
      }

      // Calculate break-even points
      for (let i = 1; i < profits.length; i++) {
        if ((profits[i] > 0 && profits[i - 1] < 0) || (profits[i] < 0 && profits[i - 1] > 0)) {
          breakEvenPoints.push(prices[i])
        }
      }

      // Set chart data and calculations
      this.chartData.labels = prices
      this.chartData.datasets[0].data = profits
      this.calculation = {
        maxProfit,
        maxLoss,
        breakEvenPoints
      }
    }
  }
})
</script>

<style scoped>
div {
  margin: 20px;
}
</style>
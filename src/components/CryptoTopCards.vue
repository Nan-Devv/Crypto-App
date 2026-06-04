<template>
  <div class="top-coins-section">
    <div class="section-title-wrapper">
      <span class="badge">Aset Utama</span>
      <p class="section-subtitle">Tiga Aset Teratas Berdasarkan Valuasi Pasar Global</p>
    </div>

    <div class="top-coins">
      <div
        class="top-card"
        v-for="(coin, index) in topThree"
        :key="coin.id"
        :class="'rank-' + (index + 1)"
      >
        <div class="card-header">
          <span class="rank-badge">RANK #{{ coin.rank }}</span>
          <span class="change-badge" :class="isPositive(coin.percent_change_24h) ? 'up' : 'down'">
            {{ isPositive(coin.percent_change_24h) ? '▲' : '▼' }} {{ Math.abs(coin.percent_change_24h) }}%
          </span>
        </div>

        <div class="card-content">
          <div class="identity">
            <h2>{{ coin.symbol }}</h2>
            <p class="name">{{ coin.name }}</p>
          </div>
          <div class="price-section">
            <span class="label">Harga USD</span>
            <div class="price">${{ formatPrice(coin.price_usd) }}</div>
          </div>
        </div>

        <div class="chart-wrapper">
          <sparkline-chart 
            :change-24h="coin.percent_change_24h" 
            :change-7d="coin.percent_change_7d" 
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SparklineChart from "./SparklineChart.vue";

export default {
  name: "CryptoTopCards",
  components: {
    SparklineChart
  },
  props: {
    cryptos: {
      type: Array,
      required: true
    }
  },
  computed: {
    topThree() {
      return this.cryptos.slice(0, 3);
    }
  },
  methods: {
    isPositive(value) {
      return parseFloat(value) >= 0;
    },
    formatPrice(price) {
      const num = parseFloat(price);
      return num.toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 4
      });
    }
  }
};
</script>

<style scoped>
.top-coins-section {
  padding: 0 16px 20px;
}
.section-title-wrapper {
  margin-bottom: 12px;
}
.badge {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1.5px;
  color: #10b981;
  background: rgba(10, 94, 2, 0.13);
  border: 1px solid rgba(114, 244, 63, 0.2);
  padding: 3px 8px;
  border-radius: 100px;
  font-family: monospace;
}
.section-subtitle {
  color: #64748b;
  font-size: 14px;
  line-height: 1.5;
}
.top-coins {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
}
.top-card {
  flex: 1 1 calc(33.33% - 11px);
  min-width: 250px;
  padding: 24px;
  border-radius: 24px;
  color: white;
  position: relative;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 200px;
  box-shadow: 0 10px 30px -10px rgba(0, 0, 0, 0.5);
  transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1), box-shadow 0.3s;
  box-sizing: border-box;
}
.top-card:hover {
  transform: translateY(-6px) scale(1.01);
  box-shadow: 0 16px 40px -12px rgba(99, 102, 241, 0.3);
}
.rank-1 {
  background: linear-gradient(135deg, #1e1b4b 0%, #311082 100%);
  border: 1px solid rgba(139, 92, 246, 0.25);
}
.rank-2 {
  background: linear-gradient(135deg, #0f172a 0%, #172554 100%);
  border: 1px solid rgba(59, 130, 246, 0.2);
}
.rank-3 {
  background: linear-gradient(135deg, #070f1e 0%, #064e3b 100%);
  border: 1px solid rgba(16, 185, 129, 0.2);
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.rank-badge {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 1px;
  color: rgba(255, 255, 255, 0.8);
  font-family: monospace;
}
.change-badge {
  font-size: 11px;
  font-weight: 700;
  padding: 4px 8px;
  border-radius: 8px;
  font-family: monospace;
}
.change-badge.up {
  background: rgba(16, 185, 129, 0.2);
  color: #34d399;
}
.change-badge.down {
  background: rgba(239, 68, 68, 0.2);
  color: #f87171;
}
.card-content {
  margin: 12px 0;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}
.identity h2 {
  margin: 0;
  font-size: 32px;
  font-weight: 850;
  letter-spacing: -1.5px;
  line-height: 1;
}
.identity .name {
  margin: 4px 0 0;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.6);
}
.price-section {
  text-align: right;
}
.price-section .label {
  font-size: 9px;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.4);
}
.price-section .price {
  font-size: 18px;
  font-weight: 800;
  font-family: monospace;
  color: #ffffff;
}
.chart-wrapper {
  margin-top: 10px;
}
@media (max-width: 768px) {
  .top-card {
    flex: 1 1 100%;
  }
}
</style>
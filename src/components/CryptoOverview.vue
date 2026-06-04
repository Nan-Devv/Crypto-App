<template>
  <div class="overview">

    <!-- Card 1: Total Aset -->
    <div class="overview-card glass">
      <div class="icon-wrapper">
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="8" cy="8" r="6"/>
          <circle cx="18" cy="18" r="4"/>
          <path d="M12 8h8M4 16h8"/>
        </svg>
      </div>
      <div class="card-info">
        <span>Tipe Kripto</span>
        <h3>{{ totalCryptos }} <small>Koin</small></h3>
        <p class="subtitle">Aset Aktif Terdaftar</p>
      </div>
    </div>

    <!-- Card 2: Status Pasar -->
    <div class="overview-card glass">
      <div class="icon-wrapper green-bg">
        <span class="pulse-indicator"></span>
      </div>
      <div class="card-info">
        <span>Status Pasar</span>
        <h3 class="green">LIVE</h3>
        <p class="subtitle">Terhubung Real-time</p>
      </div>
    </div>

    <!-- Card 3: Top Performer (Gainer Terbesar) -->
    <div class="overview-card glass">
      <div class="icon-wrapper gold-bg">
        <svg class="icon gold" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="m19 12-7-7-7 7M12 5v14"/>
        </svg>
      </div>
      <div class="card-info">
        <span>Top Gainer (24h)</span>
        <h3 v-if="topGainer">{{ topGainer.symbol }} <small class="green">+{{ topGainer.percent_change_24h }}%</small></h3>
        <h3 v-else>...</h3>
        <p class="subtitle">Kenaikan Tertinggi</p>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  name: "CryptoOverview",
  props: {
    cryptos: {
      type: Array,
      required: true
    }
  },
  computed: {
    totalCryptos() {
      return this.cryptos.length;
    },
    topGainer() {
      if (this.cryptos.length === 0) return null;
      return [...this.cryptos].sort((a, b) => {
        return parseFloat(b.percent_change_24h || 0) - parseFloat(a.percent_change_24h || 0);
      })[0];
    }
  }
};
</script>

<style scoped>
.overview {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
  padding: 8px 16px 20px;
}
.overview-card {
  display: flex;
  align-items: center;
  gap: 16px;
  background: rgba(17, 24, 39, 0.45);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  padding: 20px;
  border-radius: 20px;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s cubic-bezier(0.16, 1, 0.3, 1), border-color 0.3s;
}
.overview-card:hover {
  transform: translateY(-4px);
  border-color: rgba(99, 102, 241, 0.4);
}
.icon-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  border-radius: 14px;
  background: rgba(99, 102, 241, 0.15);
  color: #818cf8;
}
.green-bg {
  background: rgba(16, 185, 129, 0.15);
}
.gold-bg {
  background: rgba(245, 158, 11, 0.15);
}
.icon {
  width: 24px;
  height: 24px;
}
.icon.gold {
  color: #f59e0b;
}
.card-info {
  display: flex;
  flex-direction: column;
}
.card-info span {
  color: #94a3b8;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 600;
}
.card-info h3 {
  color: white;
  font-size: 20px;
  font-weight: 800;
  margin: 6px 0 2px;
  display: flex;
  align-items: baseline;
  gap: 6px;
}
.card-info h3 small {
  font-size: 13px;
  font-weight: 600;
  font-family: monospace;
}
.subtitle {
  color: #64748b;
  font-size: 12px;
}
.green {
  color: #10b981;
}
.pulse-indicator {
  display: inline-block;
  width: 10px;
  height: 10px;
  background: #10b981;
  border-radius: 50%;
  position: relative;
}
.pulse-indicator::after {
  content: '';
  position: absolute;
  top: -4px;
  left: -4px;
  right: -4px;
  bottom: -4px;
  border-radius: 50%;
  border: 2px solid #10b981;
  animation: ripple 1.6s infinite ease-out;
  opacity: 0;
}
@keyframes ripple {
  0% { transform: scale(0.6); opacity: 0.8; }
  100% { transform: scale(1.8); opacity: 0; }
}
</style>
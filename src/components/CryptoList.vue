<template>
  <div class="ledger-box">

    <!-- BAR CARIAN (SEARCH SEC) -->
    <div class="search-container">
      <div class="search-box">
        <svg class="search-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8"/>
          <path d="m21 21-4.3-4.3"/>
        </svg>
        <input
          type="text"
          :value="modelValue"
          @input="$emit('update:modelValue', $event.target.value)"
          placeholder="Cari nama koin, simbol, atau kode..."
          class="dark-input"
        />
        <button v-if="modelValue" @click="$emit('update:modelValue', '')" class="clear-btn">
          ❌
        </button>
      </div>
      <div class="search-meta" v-if="modelValue">
        Ditemukan: <strong>{{ filteredCount }}</strong> koin
      </div>
    </div>

    <!-- AREA TABLE LEDGER -->
    <div class="table-wrapper glass">
      
      <!-- Headings -->
      <div class="table-header">
        <div class="col-rank">#</div>
        <div>Coin Name</div>
        <div class="col-center">Symbol</div>
        <div class="col-right">24h Trade</div>
        <div class="col-right">Price USD</div>
        <div class="col-arrow"></div>
      </div>

      <!-- State Kosong jika hasil cari tidak ada -->
      <div v-if="filteredCount === 0" class="empty-layout">
        <div class="empty-icon">📂</div>
        <h4>Koin Tidak Ditemukan</h4>
        <p>Silakan gunakan kata kunci atau kode koin lainnya.</p>
        <button @click="$emit('update:modelValue', '')" class="reset-btn">
          Reset Pencarian
        </button>
      </div>

      <!-- Kumpulan Baris Konten -->
      <div v-else class="rows-container">
        <div 
          v-for="coin in filteredCoins" 
          :key="coin.id" 
          class="row-group"
          :class="{ 'is-expanded': expandedId === coin.id }"
        >
          <!-- Baris Utama Koin -->
          <div 
            class="coin-row" 
            @click="toggleRow(coin.id)"
          >
            <div class="col-rank font-mono">{{ coin.rank }}</div>
            
            <div class="coin-identity">
              <span class="coin-name-text">{{ coin.name }}</span>
            </div>

            <div class="col-center">
              <span class="symbol-tag">{{ coin.symbol }}</span>
            </div>

            <div class="col-right trend" :class="isPositive(coin.percent_change_24h) ? 'green' : 'red'">
              <span class="trend-indicator">{{ isPositive(coin.percent_change_24h) ? '▲' : '▼' }}</span>
              <span class="font-mono">{{ formatPercent(coin.percent_change_24h) }}%</span>
            </div>

            <div class="col-right price font-mono">
              ${{ formatPrice(coin.price_usd) }}
            </div>

            <div class="col-arrow">
              <span class="chevron" :class="{ 'rotate': expandedId === coin.id }">▼</span>
            </div>
          </div>

          <!-- Laci Detail Ekstra Koin (Accordion) -->
          <div v-if="expandedId === coin.id" class="details-accordion">
            <div class="details-grid">
              
              <div class="detail-col">
                <h5>📊 Fluktuasi Sesi</h5>
                <div class="info-row">
                  <span>Perubahan 1 Jam:</span>
                  <strong :class="isPositive(coin.percent_change_1h) ? 'green' : 'red'">
                    {{ coin.percent_change_1h }}%
                  </strong>
                </div>
                <div class="info-row">
                  <span>Perubahan 24 Jam:</span>
                  <strong :class="isPositive(coin.percent_change_24h) ? 'green' : 'red'">
                    {{ coin.percent_change_24h }}%
                  </strong>
                </div>
                <div class="info-row">
                  <span>Perubahan 7 Hari:</span>
                  <strong :class="isPositive(coin.percent_change_7d) ? 'green' : 'red'">
                    {{ coin.percent_change_7d }}%
                  </strong>
                </div>
              </div>

              <div class="detail-col">
                <h5>💼 Finansial & Likuiditas</h5>
                <div class="info-row">
                  <span>Market Cap (USD):</span>
                  <span class="font-mono">${{ formatBigNumber(coin.market_cap_usd) }}</span>
                </div>
                <div class="info-row">
                  <span>Volume Dagang 24h:</span>
                  <span class="font-mono">${{ formatBigNumber(coin.volume24) }}</span>
                </div>
                <div class="info-row">
                  <span>Name ID:</span>
                  <span class="font-mono lowcase">{{ coin.nameid }}</span>
                </div>
              </div>

              <div class="detail-col">
                <h5>🪙 Suplai Terdistribusi</h5>
                <div class="info-row">
                  <span>Suplai Beredar:</span>
                  <span class="font-mono">{{ formatBigNumber(coin.csupply) }} {{ coin.symbol }}</span>
                </div>
                <div class="info-row">
                  <span>Total Suplai:</span>
                  <span class="font-mono">{{ formatBigNumber(coin.tsupply) }} {{ coin.symbol }}</span>
                </div>
                <div class="action-btn-row">
                  <a 
                    :href="'https://www.coinlore.com/coin/' + coin.nameid" 
                    target="_blank" 
                    rel="noopener noreferrer"
                    class="view-more-btn"
                  >
                    Detail CoinLore ↗
                  </a>
                </div>
              </div>

            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
export default {
  name: "CryptoList",
  props: {
    cryptos: {
      type: Array,
      required: true
    },
    modelValue: {
      type: String,
      default: ""
    }
  },
  emits: ["update:modelValue"],
  data() {
    return {
      expandedId: null
    };
  },
  computed: {
    filteredCoins() {
      const query = this.modelValue.toLowerCase().trim();
      if (!query) return this.cryptos;
      return this.cryptos.filter(coin => 
        coin.name.toLowerCase().includes(query) ||
        coin.symbol.toLowerCase().includes(query)
      );
    },
    filteredCount() {
      return this.filteredCoins.length;
    }
  },
  methods: {
    toggleRow(id) {
      this.expandedId = this.expandedId === id ? null : id;
    },
    isPositive(value) {
      return parseFloat(value) >= 0;
    },
    formatPrice(price) {
      const val = parseFloat(price);
      return val.toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 6
      });
    },
    formatPercent(percent) {
      const val = parseFloat(percent);
      return (val >= 0 ? "+" : "") + val.toFixed(2);
    },
    formatBigNumber(numStr) {
      const num = parseFloat(numStr || 0);
      return num.toLocaleString(undefined, {
        maximumFractionDigits: 0
      });
    }
  }
};
</script>

<style scoped>
.ledger-box {
  margin: 0 16px 24px;
}
.search-container {
  margin-bottom: 20px;
}
.search-box {
  display: flex;
  align-items: center;
  position: relative;
  background: rgba(15, 23, 42, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 18px;
  overflow: hidden;
  padding: 2px 14px;
}
.search-icon {
  width: 20px;
  height: 20px;
  color: #64748b;
  margin-right: 12px;
}
.dark-input {
  flex: 1;
  background: transparent;
  border: none;
  height: 54px;
  color: white;
  font-size: 14.5px;
  outline: none;
}
.dark-input::placeholder {
  color: #475569;
}
.clear-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 14px;
  padding: 6px;
}
.search-meta {
  font-size: 12px;
  color: #64748b;
  margin-top: 8px;
  margin-left: 6px;
}
.table-wrapper {
  background: rgba(17, 24, 39, 0.4);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 22px;
  overflow: hidden;
}
.table-header {
  display: grid;
  grid-template-columns: 0.5fr 2.5fr 1fr 1.5fr 1.8fr 0.4fr;
  padding: 18px 20px;
  color: #64748b;
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-family: monospace;
  background: rgba(10, 15, 30, 0.4);
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}
.coin-row {
  display: grid;
  grid-template-columns: 0.5fr 2.5fr 1fr 1.5fr 1.8fr 0.4fr;
  padding: 18px 20px;
  color: white;
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);
  align-items: center;
  transition: background 0.25s, transform 0.2s;
  cursor: pointer;
}
.coin-row:hover {
  background: rgba(255, 255, 255, 0.03);
}
.row-group.is-expanded {
  background: rgba(99, 102, 241, 0.03);
}
.col-rank {
  color: #475569;
  font-weight: 600;
  font-size: 12px;
}
.coin-identity {
  display: flex;
  align-items: center;
  font-weight: 700;
  font-size: 14px;
}
.symbol-tag {
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.2);
  color: #a5b4fc;
  font-family: monospace;
  font-size: 11px;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 6px;
}
.col-center { text-align: center; }
.col-right { text-align: right; }
.col-right.price {
  font-size: 14px;
  font-weight: 700;
}
.trend {
  font-size: 13px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 4px;
}
.trend-indicator { font-size: 10px; }
.green { color: #10b981; }
.red { color: #ef4444; }
.col-arrow {
  display: flex;
  justify-content: flex-end;
  color: #475569;
}
.chevron {
  font-size: 10px;
  transition: transform 0.3s ease;
}
.chevron.rotate {
  transform: rotate(-180deg);
  color: #818cf8;
}
.details-accordion {
  background: rgba(10, 15, 30, 0.5);
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  padding: 24px;
}
.details-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 24px;
}
.detail-col h5 {
  margin: 0 0 12px;
  color: #94a3b8;
  font-size: 11px;
  text-transform: uppercase;
}
.info-row {
  display: flex;
  justify-content: space-between;
  font-size: 12.5px;
  margin-bottom: 8px;
  color: #64748b;
}
.info-row strong, .info-row span {
  font-family: monospace;
}
.info-row strong.green { color: #10b981; }
.info-row strong.red { color: #ef4444; }
.lowcase { text-transform: lowercase; }
.action-btn-row {
  margin-top: 14px;
  display: flex;
}
.view-more-btn {
  display: inline-block;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  color: #a5b4fc;
  text-decoration: none;
  font-size: 12px;
  font-weight: 600;
  padding: 8px 16px;
  border-radius: 10px;
  width: 100%;
  text-align: center;
  box-sizing: border-box;
}
.view-more-btn:hover {
  background: #4f46e5;
  color: white;
  border-color: #4f46e5;
}
.empty-layout {
  padding: 60px 20px;
  text-align: center;
}
.empty-icon {
  font-size: 40px;
  margin-bottom: 12px;
}
.empty-layout h4 {
  color: white;
  margin: 0 0 6px;
  font-size: 16px;
}
.empty-layout p {
  color: #64748b;
  font-size: 13px;
}
.reset-btn {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border: none;
  font-size: 12px;
  font-weight: 600;
  padding: 8px 20px;
  border-radius: 10px;
  cursor: pointer;
  margin-top: 12px;
}
@media (max-width: 640px) {
  .table-header {
    grid-template-columns: 0.5fr 2fr 1fr 1.5fr;
  }
  .table-header .col-arrow, .table-header .col-right:nth-child(5) {
    display: none;
  }
  .coin-row {
    grid-template-columns: 0.5fr 2fr 1fr 1.5fr;
  }
  .coin-row .col-arrow, .coin-row .col-right.price {
    display: none;
  }
}
</style>
//npm install -g @ionic/cli
//ionic start crypto-app blank --type vue
//cd crypto-app
//npm install axios
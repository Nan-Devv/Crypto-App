<template>
  <ion-page>
    <ion-content fullscreen class="background">

      <!-- HEADER UTAMA -->
      <div class="header">
        <div class="header-inner">
          <div class="active-pulse">
            <span class="pulse-dot"></span>
            <span class="pulse-text">Live Ticker Synced</span>
          </div>
          <h1>Cryptocurrency Prices</h1>
          <p>Daftar Harga Aset Kripto Teraktual Dihitung Berdasarkan Total Market Kapitalisasi Pasar Global.</p>
        </div>

        <!-- Tombol Refresh Interaktif -->
        <button 
          @click="loadCrypto" 
          :disabled="loading" 
          class="custom-refresh-btn"
        >
          <span :class="{ 'spinning': loading }" class="refresh-symbol">↻</span>
          <span>Perbarui Data</span>
        </button>
      </div>

      <!-- REFRESHER PULL-DOWN (Bawaan Ionic) -->
      <ion-refresher slot="fixed" @ionRefresh="refreshData">
        <ion-refresher-content />
      </ion-refresher>

      <!-- SPINNER LOADING UTAMA -->
      <div v-if="loading && cryptos.length === 0" class="loading-state">
        <ion-spinner name="crescent"></ion-spinner>
        <p>Menghubungkan jaringan CoinLore...</p>
      </div>

      <!-- KONTEN UTAMA JIKA DATA BERHASIL DIUNDUH -->
      <div v-else class="dashboard-body">
        
        <!-- BAGIAN 1: BENTO OVERVIEW -->
        <crypto-overview :cryptos="cryptos" />

        <!-- BAGIAN 2: SHOWCASE TIGA BESAR (TOP 3) -->
        <crypto-top-cards :cryptos="cryptos" />

        <!-- BAGIAN 3: LEDGER INTEGRAL & PENCARIAN -->
        <crypto-list 
          v-model="search" 
          :cryptos="cryptos" 
        />

      </div>

    </ion-content>
  </ion-page>
</template>

<script>
import axios from "axios";
// Mengimpor sub-komponen terpisah
import CryptoOverview from "../components/CryptoOverview.vue";
import CryptoTopCards from "../components/CryptoTopCards.vue";
import CryptoList from "../components/CryptoList.vue";
import {
  IonPage,
  IonContent,
  IonRefresher,
  IonRefresherContent,
  IonSpinner
} from '@ionic/vue'

export default {
  name: "HomePage",
  components: {
    IonPage,
    IonContent,
    IonRefresher,
    IonRefresherContent,
    IonSpinner,
    CryptoOverview,
    CryptoTopCards,
    CryptoList
  },
  data() {
    return {
      cryptos: [],  // Menyimpan raw data koin dari feed API
      search: "",   // String filter pencarian global
      loading: true // Menyala ketika pertama kali boot up halaman
    };
  },
  mounted() {
    this.loadCrypto();
  },
  methods: {
    // Fungsi sinkronisasi data ticker dari API CoinLore
    async loadCrypto() {
      this.loading = true;
      try {
        const response = await axios.get("https://api.coinlore.net/api/tickers/");
        if (response.data && response.data.data) {
          this.cryptos = response.data.data;
        }
      } catch (error) {
        console.error("Kesalahan Sinkronisasi Feed CoinLore:", error);
      } finally {
        this.loading = false;
      }
    },
    // Handler penarik layar (pull down event) bawaan Ionic
    async refreshData(event) {
      await this.loadCrypto();
      event.target.complete();
    }
  }
};
</script>

<style scoped>
.background {
  --background: #070b13;
}
.header {
  padding: 36px 20px 20px;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  flex-wrap: wrap;
  gap: 16px;
  max-width: 1200px;
  margin: 0 auto;
}
.header-inner {
  max-width: 600px;
}
.active-pulse {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: rgba(16, 185, 129, 0.1);
  border: 1px solid rgba(16, 185, 129, 0.2);
  padding: 4px 10px;
  border-radius: 100px;
  margin-bottom: 12px;
}
.pulse-dot {
  width: 6px;
  height: 6px;
  background: #10b981;
  border-radius: 50%;
  animation: static-ping 1.4s infinite;
}
.pulse-text {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  color: #10b981;
  font-family: monospace;
}
.header h1 {
  color: white;
  font-size: clamp(20px,5vw,36px);
  font-weight: 850;
  margin: 0;
  letter-spacing: -1.5px;
  line-height: 1.1;

  overflow: hidden;
  white-space: nowrap;
  border-right: 3px solid white;
  width: 0;
  animation: typing 9s steps(30, end) infinite,
             blink 0.8s step-end infinite;
}

@keyframes typing {
  0% {
    width: 0;
  }
  40% {
    width: 100%;
  }
  40% {
    width: 59%;
  }
  100% {
    width: 0;
  }
}

@keyframes blink {
  50% {
    border-color: transparent;
  }
}

.header p {
  color: #64748b;
  margin-top: 10px;
  font-size: 14px;
  line-height: 1.5;
}
.custom-refresh-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: white;
  padding: 12px 20px;
  border-radius: 14px;
  font-size: 13.5px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.2s, border-color 0.2s;
}
.custom-refresh-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(99, 102, 241, 0.4);
}
.custom-refresh-btn:active:not(:disabled) {
  transform: scale(0.97);
}
.refresh-symbol {
  font-size: 16px;
  display: inline-block;
}
.spinning {
  animation: roll 1.2s infinite linear;
}
@keyframes roll {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
@keyframes static-ping {
  0% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.5); opacity: 0.5; }
  100% { transform: scale(1); opacity: 1; }
}
.dashboard-body {
  max-width: 1200px;
  margin: 0 auto;
}
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 20px;
}
.loading-state p {
  color: #64748b;
  font-size: 13px;
  margin-top: 14px;
  font-family: monospace;
}
ion-spinner {
  transform: scale(1.6);
  --color: #818cf8;
}
@media (max-width: 768px) {
  .header {
    padding: 30px 16px 16px;
    flex-direction: column;
    align-items: flex-start;
  }
  .custom-refresh-btn {
    width: 100%;
    justify-content: center;
  }
}
</style>
<template>
  <div class="sparkline-container">
    <svg class="sparkline-svg" viewBox="0 0 100 40" preserveAspectRatio="none">
      <defs>
        <!-- Efek Gradasi Area di bawah Garis Grafik -->
        <linearGradient :id="gradientId" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" :stop-color="color" stop-opacity="0.2" />
          <stop offset="100%" :stop-color="color" stop-opacity="0.0" />
        </linearGradient>
      </defs>

      <!-- Menggambar area gradasi di bawah grafik -->
      <path :d="areaPath" :fill="`url(#${gradientId})`" />

      <!-- Menggambar garis utama grafik -->
      <path :d="linePath" fill="none" :stroke="color" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />

      <!-- Titik Cahaya Pulsa Indikator di ujung kanan grafik -->
      <circle :cx="lastPoint.x" :cy="lastPoint.y" r="3" :fill="color" class="pulse-dot" />
    </svg>
  </div>
</template>

<script>
export default {
  name: "SparklineChart",
  props: {
    change24h: {
      type: [String, Number],
      required: true
    },
    change7d: {
      type: [String, Number],
      default: 0
    }
  },
  data() {
    return {
      gradientId: 'sparkline-grad-' + Math.random().toString(36).substr(2, 9)
    };
  },
  computed: {
    isPositive() {
      return parseFloat(this.change24h) >= 0;
    },
    color() {
      return this.isPositive ? "#10b981" : "#ef4444";
    },
    points() {
      const c24 = parseFloat(this.change24h);
      const c7 = parseFloat(this.change7d);
      const values = [40];
      const step24 = c24 / 8;
      const step7 = c7 / 8;

      for (let i = 1; i <= 7; i++) {
        // Formulasi matematika sinus/cosinus (noise) agar kurva grafik terlihat meliuk natural
        const noise = (Math.sin(i * 1.6) * 6) + (Math.cos(i * 2.1) * 3);
        const trend = (step7 * i * 0.4) + (step24 * (i - 3) * 0.6);
        const calculatedValue = 40 + (trend * 4.5) + noise;
        values.push(Math.max(10, Math.min(90, calculatedValue)));
      }

      const width = 100;
      const height = 40;
      const segmentX = width / (values.length - 1);

      return values.map((val, index) => {
        const x = index * segmentX;
        const y = height - (val * (height / 100));
        return { x, y };
      });
    },
    lastPoint() {
      const pts = this.points;
      return pts[pts.length - 1] || { x: 100, y: 20 };
    },
    linePath() {
      return this.points.map((p, index) => `${index === 0 ? 'M' : 'L'} ${p.x.toFixed(1)} ${p.y.toFixed(1)}`).join(' ');
    },
    areaPath() {
      if (this.points.length === 0) return '';
      const start = `M 0 40`;
      const pointsStr = this.points.map(p => `L ${p.x.toFixed(1)} ${p.y.toFixed(1)}`).join(' ');
      const end = `L 100 40 Z`;
      return `${start} ${pointsStr} ${end}`;
    }
  }
};
</script>

<style scoped>
.sparkline-container {
  width: 100%;
  height: 42px;
  display: flex;
  align-items: center;
}
.sparkline-svg {
  width: 100%;
  height: 100%;
  overflow: visible;
}
.pulse-dot {
  animation: pulse 1.8s infinite ease-in-out;
}
@keyframes pulse {
  0% { r: 3px; opacity: 1; }
  50% { r: 5.5px; opacity: 0.6; }
  100% { r: 3px; opacity: 1; }
}
</style>
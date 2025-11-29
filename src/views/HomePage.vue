<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar class="toolbar">
        <ion-title>Crypto Tracker</ion-title>
        <ion-buttons slot="end">
          <ion-button
            color="secondary"
            @click="onRefreshClick"
            :disabled="isLoading"
          >
            <ion-icon :icon="refreshOutline" slot="start" />
            Refresh
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="content">
      <ion-refresher slot="fixed" @ionRefresh="handlePullToRefresh">
        <ion-refresher-content
          pulling-text="Pull to refresh"
          refreshing-spinner="crescent"
          refreshing-text="Updating prices..."
        />
      </ion-refresher>

      <div class="hero">
        <div>
          <p class="subtitle">Data from Coinlore API</p>
        </div>
        <ion-spinner v-if="isLoading" name="crescent" class="hero-spinner" />
      </div>

      <section class="list-area">
        <div v-if="error" class="state error">{{ error }}</div>
        <div v-else-if="isLoading && coins.length === 0" class="state">
          <ion-spinner name="crescent" />
          <p>Loading prices...</p>
        </div>
        <div v-else-if="!coins.length" class="state">
          <p>No data yet. Try refreshing.</p>
        </div>

        <div v-else class="cards">
          <article v-for="coin in coins" :key="coin.id" class="coin-card">
            <div class="coin-left">
              <div class="rank">#{{ coin.rank }}</div>
              <div class="icon">
                <img
                  v-if="!failedIcons[coin.symbol]"
                  :src="coinIcon(coin.symbol)"
                  :alt="coin.symbol"
                  loading="lazy"
                  @error="markIconFailed(coin.symbol)"
                />
                <span v-else class="fallback">{{ coin.symbol[0] }}</span>
              </div>
              <div class="name">
                {{ coin.name }}
                <span class="symbol">({{ coin.symbol }})</span>
              </div>
            </div>
            <div class="coin-right">
              <div class="price">${{ formatPrice(coin.price_usd) }}</div>
            </div>
          </article>
        </div>
      </section>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import {
  IonButtons,
  IonButton,
  IonContent,
  IonHeader,
  IonIcon,
  IonPage,
  IonRefresher,
  IonRefresherContent,
  IonSpinner,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";
import { refreshOutline } from "ionicons/icons";
import axios from "axios";

type Coin = {
  id: string;
  rank: number;
  name: string;
  symbol: string;
  price_usd: string;
};

const coins = ref<Coin[]>([]);
const isLoading = ref(false);
const error = ref("");
const failedIcons = ref<Record<string, boolean>>({});

const fetchCoins = async () => {
  isLoading.value = true;
  error.value = "";
  try {
    const { data } = await axios.get("https://api.coinlore.net/api/tickers/");
    coins.value = Array.isArray(data?.data) ? data.data : [];
  } catch (err) {
    error.value = axios.isAxiosError(err)
      ? err.message || "Request failed"
      : err instanceof Error
      ? err.message
      : "Unknown error";
  } finally {
    isLoading.value = false;
  }
};

const handlePullToRefresh = async (event: CustomEvent) => {
  await fetchCoins();
  event.detail.complete();
};

const onRefreshClick = () => {
  fetchCoins();
};

const formatPrice = (price: string) =>
  Number(price).toLocaleString("en-US", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });

const coinIcon = (symbol: string) =>
  `https://assets.coincap.io/assets/icons/${symbol.toLowerCase()}@2x.png`;

const markIconFailed = (symbol: string) => {
  failedIcons.value = { ...failedIcons.value, [symbol]: true };
};

onMounted(fetchCoins);
</script>

<style scoped>
:global(body) {
  background: radial-gradient(circle at 20% 20%, #1b1e2b, #0c0f18 60%);
}

.content {
  --background: transparent;
  padding: 16px;
}

.toolbar {
  --background: rgba(12, 15, 24, 0.85);
  --color: #ffffff;
  backdrop-filter: blur(6px);
}

.hero {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 4px 4px 16px;
  color: #f1f5ff;
}

.eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-size: 11px;
  color: #9fb4ff;
  margin: 0 0 4px;
}

.hero h1 {
  margin: 0;
  font-size: 24px;
}

.subtitle {
  margin: 6px 0 0;
  color: #c2c7d6;
}

.hero-spinner {
  width: 28px;
  height: 28px;
}

.list-area {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #d7dcf1;
  padding: 20px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 12px;
}

.state.error {
  color: #f5a8b8;
  border-color: rgba(245, 168, 184, 0.4);
}

.cards {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding-bottom: 24px;
}

.coin-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 16px;
  background: linear-gradient(135deg, #131722, #0f121d);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.05);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.35);
}

.coin-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.icon {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #2f3c54, #0f1726);
  border: 1px solid rgba(255, 255, 255, 0.08);
  display: grid;
  place-items: center;
  overflow: hidden;
}

.icon img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.fallback {
  color: #d7e2ff;
  font-weight: 700;
  font-size: 14px;
}

.rank {
  width: 38px;
  height: 38px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.08);
  display: grid;
  place-items: center;
  color: #ffc34d;
  font-weight: 700;
}

.name {
  color: #f7f8fc;
  font-weight: 600;
  font-size: 15px;
}

.symbol {
  color: #9ab0d0;
  font-weight: 500;
}

.coin-right {
  text-align: right;
  color: #e9edff;
}

.price {
  font-weight: 700;
  font-size: 17px;
}

.muted {
  color: #9ab0d0;
  margin-top: 4px;
}

@media (max-width: 480px) {
  .hero h1 {
    font-size: 20px;
  }

  .coin-card {
    padding: 12px 14px;
  }
}
</style>

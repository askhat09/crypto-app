<template>
  <div class="app-wrapper container">
    <div class="container">
      <section>
        <div class="flex">
          <div class="ticker">
            <label for="wallet" class="ticker-label">Тикер</label>
            <div class="ticker-input_wrapper">
              <input
                v-model="ticker"
                @keydown.enter="add"
                @input="reset"
                type="text"
                name="wallet"
                id="wallet"
                class="ticker-input"
                placeholder="Например DOGE"
                autocomplete="off"
              />
            </div>
            <div v-if="matchedTickers.length > 0" class="flex tags">
              <span
                v-for="matchedTicker of matchedTickers.slice(0, 4)"
                :key="matchedTicker"
                class="tag"
              >
                {{ matchedTicker }}
              </span>
            </div>
            <div v-if="isTickerAlreadyAdded" class="ticker-error-message">
              Такой тикер уже добавлен
            </div>
          </div>
        </div>
        <button @click="add" type="button" class="ticker-add-button">
          <svg
            class="-ml-0.5 mr-2 h-6 w-6"
            xmlns="http://www.w3.org/2000/svg"
            width="30"
            height="30"
            viewBox="0 0 24 24"
            fill="#ffffff"
          >
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>
      <template v-if="tickers.length">
        <hr class="tickers-hr" />
        <dl class="tickers-wrapper">
          <div
            v-for="t of tickers"
            :key="t.name"
            @click="select(t)"
            :class="{
              'ticker-item-border': sel === t
            }"
            class="tickers-item"
          >
            <div class="tickers-item-info">
              <dt class="tickers-item-info-label">{{ t.name }} - USD</dt>
              <dd class="tickers-item-info-price">
                {{ t.price }}
              </dd>
            </div>
            <div class="tickers-item-border"></div>
            <button @click.stop="deleteTicker(t)" class="tickers-item-button">
              <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path></svg
              >Удалить
            </button>
          </div>
        </dl>
        <hr class="tickers-hr" />
        <section v-if="sel" class="ticker-graphic">
          <h3 class="ticker-graphic-heading">{{ sel.name }} - USD</h3>
          <div class="ticker-graphic-info">
            <div
              v-for="(bar, idx) in normalizeGraph()"
              :key="idx"
              :style="{ height: `${bar}%` }"
              class="ticker-graphic-info-item"
            ></div>
          </div>
          <button
            @click="sel = null"
            type="button"
            class="ticker-graphic-button"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              xmlns:xlink="http://www.w3.org/1999/xlink"
              xmlns:svgjs="http://svgjs.com/svgjs"
              version="1.1"
              width="30"
              height="30"
              x="0"
              y="0"
              viewBox="0 0 511.76 511.76"
              style="enable-background: new 0 0 512 512"
              xml:space="preserve"
            >
              <g>
                <path
                  d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                  fill="#718096"
                  data-original="#000000"
                ></path>
              </g>
            </svg>
          </button>
        </section>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",

  data() {
    return {
      ticker: "",
      tickers: [],
      coinlist: null,
      sel: null,
      graph: [],
      isTickerAlreadyAdded: false,
      matchedTickers: []
    };
  },

  async mounted() {
    const t = await fetch(
      `https://min-api.cryptocompare.com/data/all/coinlist?api_key=6b99cd9d745149f02adeeba1d29fda9f1731c711a4aa078a94addf781a379630`
    );
    const response = await t.json();
    this.coinlist = response.Data;
  },

  methods: {
    add() {
      const newTicker = {
        name: this.ticker.toUpperCase(),
        price: null
      };

      this.tickers.forEach(t => {
        if (t.name === this.ticker.toUpperCase()) {
          this.isTickerAlreadyAdded = true;
        }
      });

      if (this.isTickerAlreadyAdded) {
        return;
      }

      this.tickers.push(newTicker);
      this.ticker = "";

      setInterval(async () => {
        const f = await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${newTicker.name}&tsyms=USD&key=6b99cd9d745149f02adeeba1d29fda9f1731c711a4aa078a94addf781a379630`
        );
        const data = await f.json();
        this.tickers.find(t => t.name === newTicker.name).price =
          data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);

        if (this.sel?.name === newTicker.name) {
          this.graph.push(data.USD);
        }
      }, 3000);
    },

    reset() {
      this.isTickerAlreadyAdded = false;
      this.matchedTickers = [];
      Object.keys(this.coinlist).forEach(t => {
        if (t.includes(this.ticker.toUpperCase())) {
          this.matchedTickers.push(t);
        }
      });
      if (!this.ticker) {
        this.matchedTickers = [];
      }
    },

    deleteTicker(ticker) {
      this.tickers = this.tickers.filter(t => t != ticker);
    },

    normalizeGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);

      return this.graph.map(
        price => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
    },

    select(ticker) {
      this.sel = ticker;
      this.graph = [];
    }
  }
};
</script>

<style src="./app.css"></style>

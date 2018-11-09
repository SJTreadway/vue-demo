<template>
  <div>
    <h1>Crypto Compare</h1>
    <div class="row">
      <div class="jumbotron col-xs-offset-2 col-xs-8">
        <p>
          This website indexes the top 10 cryptocurrencies by market cap (how
          much the sum of all coins is collectively worth), and gives you an easy
          way to compare cryptocurrency performance and rank over the last week.
        </p>
      </div>
      <table class="table table-hover">
        <thead>
          <tr>
            <td>Rank</td>
            <td>Name</td>
            <td>Symbol</td>
            <td>Price (USD)</td>
            <td>1H</td>
            <td>1D</td>
            <td>1W</td>
            <td>Market Cap (USD)</td>
          </tr>
        </thead>
        <tbody>
          <tr v-for="coin in coins" :key="coin.id">
            <td>{{ coin.rank }}</td>
            <td><img :src="getCoinImage(coin.symbol)" alt="">{{ coin.name }}</td>
            <td>{{ coin.symbol }}</td>
            <td>{{ coin.price_usd | currency }}</td>
            <td :style="getColor(coin.percent_change_1h)">
              <span v-if="coin.percent_change_1h > 0">+</span>{{ coin.percent_change_1h }}%
            </td>
            <td :style="getColor(coin.percent_change_24h)">
              <span v-if="coin.percent_change_24h > 0">+</span>{{ coin.percent_change_24h }}%
            </td>
            <td :style="getColor(coin.percent_change_7d)">
              <span v-if="coin.percent_change_7d > 0">+</span>{{ coin.percent_change_7d }}%
            </td>
            <td>{{ coin.market_cap_usd | currency }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
// The API we're using for grabbing metadata about each cryptocurrency
// The service can be found at: https://www.cryptocompare.com/api/
const CRYPTOCOMPARE_API_URI = 'https://min-api.cryptocompare.com'

const CRYPTOCOMPARE_IMAGE_API_URI = 'https://www.cryptocompare.com'

// The API we're using for grabbing cryptocurrency prices.  The service can be
// found at: https://coinmarketcap.com/api/
const COINMARKETCAP_API_URI = 'https://api.coinmarketcap.com'

// The amount of milliseconds (ms) after which we should update our currency
// charts.
const UPDATE_INTERVAL = 60 * 1000

export default {
  name: 'crypto',
  data () {
    return {
      coins: [],
      coinData: {}
    }
  },
  created () {
    this.getCoinData()
    this.refresh()
  },
  methods: {
    /**
     * Load up all cryptocurrency data.  This data is used to find what logos
     * each currency has, so we can display things in a friendly way.
    */
    getCoinData: function () {
      axios.get(CRYPTOCOMPARE_API_URI + '/data/all/coinlist')
        .then(resp => {
          this.coinData = resp.data.Data
          this.getCoins()
        })
        .catch(error => {
          this.getCoins()
          console.error('error receiving coin data: ', error)
        })
    },

    /**
     * Get the top 10 cryptocurrencies by value.  This data is refreshed each 5
     * minutes by the backing API service.
    */
    getCoins: function () {
      axios.get(COINMARKETCAP_API_URI + '/v1/ticker/?limit=10')
        .then(resp => {
          this.coins = resp.data
        })
        .catch(err => console.error('error retriving coins: ', err))
    },

    /**
     * Given a cryptocurrency ticket symbol, return the currency's logo
     * image.
    */
    getCoinImage: function (symbol) {
      return CRYPTOCOMPARE_IMAGE_API_URI + this.coinData[symbol].ImageUrl
    },
    /**
     * Return a CSS color (either red or green) depending on whether or
     * not the value passed in is negative or positive.
    */
    getColor: function (num) {
      return num > 0 ? 'color: green' : 'color: red'
    },
    /**
     * Once the page has been loaded and all of our app stuff is working, we'll
     * start polling for new cryptocurrency data every minute.
     */
    refresh: function () {
      setInterval(() => {
        this.getCoins()
      }, UPDATE_INTERVAL)
    }
  }
}
</script>

<style>
h1 {
  text-align: center;
}

td img {
  width: 25px;
}

.jumbotron p {
  font-size: 1.2em;
}

.jumbotron {
  margin-top: 5em;
  margin-bottom: 5em;
}
</style>

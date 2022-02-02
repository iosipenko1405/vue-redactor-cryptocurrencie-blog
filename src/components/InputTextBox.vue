<template>
  <div class="col">
    <form class="box">
      <textarea
        placeholder="Please add your text"
        v-model="textToParse"
      ></textarea>
    </form>
    <transition v-if="errorsList.length" name="fade" mode="out-in">
      <div class="error-box">
        <div
          v-for="(error, index) in errorsList"
          :key="`err-${index}-${error}`"
          class="error-msg"
        >
          {{ error }}
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  props: {
    coinsList: {
      required: true,
      type: Array,
    },
  },
  data() {
    return {
      isLoading: false,
      errorsList: [],
      usedCoinsList: new Map(),
      textToParse: '',
      updateEach: 30000,
      regExp: /{{\s*(\Name|Exchange)\/([a-z]+)\s*}}/gi,
    }
  },
  watch: {
    async textToParse() {
      const resultData = await this.parseText()
      this.$emit('textParsed', resultData)
      this.isLoading = false
    },
    isLoading() {
      this.$emit('loadingChanged', this.isLoading)
    },
  },
  emits: ['textParsed', 'loadingChanged'],
  methods: {
    addRequestToExchanges(exchangeSet, symbol) {
      if (this.usedCoinsList.has(symbol) &&
          (!this.usedCoinsList.get(symbol).exchangeRate ||
            Date.now() - this.usedCoinsList.get(symbol).timeUpdated >
              this.updateEach)) {
                exchangeSet.add({
                  symbol: symbol,
                  id: this.usedCoinsList.get(symbol).id,
                })
              }
    },
    async parseText() {
      const requestExchangeSet = new Set()
      let item = []
      this.errorsList = []

      while (item = this.regExp.exec(this.textToParse)) {
        let [_, method, symbol] = [...item]

        if (!this.usedCoinsList.has(symbol)) {
          const curCoin = this.coinsList.find((coin) => coin.symbol === symbol)

          if (curCoin) {
            this.usedCoinsList.set(symbol, {
              id: curCoin.id,
              name: curCoin.name,
            })
          } else {
            this.errorsList.push(
              `Sorry, we don't have any information for ${symbol}`,
            )
          }
        }

        if (method === 'Exchange') {
          this.addRequestToExchanges(requestExchangeSet, symbol);
        }
      }

      if (requestExchangeSet.size) {
        this.isLoading = true
        await this.loadExchangeRates(Array.from(requestExchangeSet))
      }

      const resultData = this.textToParse.replaceAll(
        this.regExp,
        (result, method, symbol) => {
          return this.usedCoinsList.has(symbol)
            ? method === 'Name'
              ? this.usedCoinsList.get(symbol).name
              : this.usedCoinsList.get(symbol).exchangeRate
            : result
        },
      )

      return resultData
    },
    async loadExchangeRates(requestExchangeArr) {
      const limit = 9
      const delayTime = 1000 / limit
      const promisesRequest = []

      const sleep = (ms) =>
        new Promise((resolve) =>
          setTimeout(() => {
            resolve()
          }, ms),
        )

      for (let i = 0; i < requestExchangeArr.length; i++) {
        promisesRequest.push(this.getExchangeRate(requestExchangeArr[i]))
        await sleep(delayTime)
      }

      return Promise.all(promisesRequest)
    },
    getExchangeRate(coin) {
      return fetch(
        `https://api.coinpaprika.com/v1/coins/${coin.id}/ohlcv/today`,
      )
        .then(async (response) => {
          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`)
          }

          return response.json()
        })
        .then((data) => {
          this.usedCoinsList.get(coin.symbol).exchangeRate = `${data[0].close.toFixed(3)} USD`
          this.usedCoinsList.get(coin.symbol).timeUpdated = Date.now()
        })
        .catch((error) => {
          this.errorsList.push(error)
        })
    },
  },
}
</script>

<style lang="scss" scoped>
textarea {
  padding: 0;
  outline: none;
  width: 100%;
  min-height: 100%;
  resize: none;
  border: none;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 14px;
}

.error-box {
  margin: 20px 0 0;
}
</style>

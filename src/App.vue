<template>
  <h1>Redactor of a cryptocurrencie's blog</h1>

  <div class="wrapper">
    <span v-if="isLoading" class="svg-icon icon-loading">
      <img :src="require('./assets/icons/icon-loading.svg')" />
    </span>
    <transition v-if="isError" name="fade" mode="out-in">
      <div class="error-box text-center">
        <div class="error-msg">Something happened. Please try again</div>
        <div>
          <span class="btn" @click="getCoinsList">Retry</span>
        </div>
      </div>
    </transition>
    <transition-group v-if="originalCoinList.length" name="fade" mode="out-in">
      <input-text-box
        :coinsList="originalCoinList"
        @textParsed="onChangeOuterText"
        @loadingChanged="onOuterLoadingChanged"
        key="1"
      ></input-text-box>
      <output-text-box
        :outerText="outerText"
        :isLoading="isOuterLoading"
        key="2"
      ></output-text-box>
    </transition-group>
  </div>
</template>

<script>
import InputTextBox from './components/InputTextBox.vue'
import OutputTextBox from './components/OutputTextBox.vue'

export default {
  components: {
    InputTextBox,
    OutputTextBox,
  },
  data() {
    return {
      isLoading: false,
      isOuterLoading: false,
      outerText: '',
      originalCoinList: [],
      isError: false,
    }
  },
  created() {
    this.getCoinsList()
  },
  methods: {
    onChangeOuterText(value) {
      this.outerText = value
    },
    onOuterLoadingChanged(value) {
      this.isOuterLoading = value
    },
    getCoinsList() {
      this.isLoading = true
      this.isError = false

      fetch('https://api.coinpaprika.com/v1/coins')
        .then((response) => {
          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`)
          }

          return response.json()
        })
        .then((data) => {
          this.originalCoinList = data
        })
        .catch(() => {
          this.isError = true
        })
        .finally(() => (this.isLoading = false))
    },
  },
}
</script>

<style lang="scss" scoped>
.wrapper {
  display: flex;
  justify-content: space-between;
  width: 1000px;
  max-width: 100%;
  position: relative;
  padding: 15px;
  box-sizing: border-box;
}

h1 {
  font-size: 30px;
  line-height: 120%;
  text-align: center;
  padding: 40px 0 15px;
}

.col {
  flex: 0 1 auto;
  width: 470px;
  max-width: 100%;
}

.error-box {
  margin: auto;
  display: flex;
  flex-direction: column;
}

.error-msg {
  font-size: 20px;
  margin-block: 30px;
  padding: 0 0 10px;
}

.btn {
  border: 2px solid green;
  padding: 10px 20px;
  display: inline-block;
  border-radius: 10px;
  color: green;
  cursor: pointer;
  font-weight: bold;
  min-width: 125px;
}

@media screen and (max-width: 768px) {
  .wrapper {
    flex-direction: column;
  }

  .col {
    margin: 0 0 15px;
  }
}
</style>

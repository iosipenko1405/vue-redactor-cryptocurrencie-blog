<template>
  <div class="col">
    <div class="box box-result">
      <span v-if="isLoading" class="svg-icon icon-loading">
        <img :src="require('../assets/icons/icon-loading.svg')" />
      </span>
      <div>{{ outerText }}</div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    outerText: {
      type: String,
      required: true
    },
    isLoading: {
      type: Boolean,
      required: false
    }
  }
}
</script>

<style lang="scss" scoped>

  .box-result {
    width: 100%;
    position: relative;
  }
</style>
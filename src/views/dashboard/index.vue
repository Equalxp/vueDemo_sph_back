<template>
  <div>
    <Card></Card>
    <!-- <Sale></Sale> -->
  </div>
</template>

<script>
import Card from './card/index.vue'
import Sale from './sale/index.vue'
export default {
  name: 'Dashboard',
  components:{
    Card,
    Sale

  }
}
</script>

<style lang="scss" scoped>
</style>

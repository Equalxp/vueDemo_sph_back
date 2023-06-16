<template>
  <a :title="title" style="margin:10px">
    <el-button v-bind="$attrs" v-on="$listeners"></el-button>
    <!-- $attrs拿到hint-button里面的属性 -->
    <!-- $listener拿到方法 -->
  </a>
</template>

<script>
export default {
  name:'HintButton',
  props:['title'],
  mounted() {

  },
}
</script>

<style>

</style>
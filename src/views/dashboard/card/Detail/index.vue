<template>
  <div>
    <div class="card-header">
      <span>{{ title }}</span>
      <svg
        t="1681525944319"
        class="icon"
        viewBox="0 0 1024 1024"
        version="1.1"
        xmlns="http://www.w3.org/2000/svg"
        p-id="1664"
        width="16"
        height="16"
      >
        <path
          d="M512 64C264.58 64 64 264.58 64 512s200.58 448 448 448 448-200.58 448-448S759.42 64 512 64z m0 824c-207.66 0-376-168.34-376-376s168.34-376 376-376 376 168.34 376 376-168.34 376-376 376z"
          fill="#1A1A1A"
          p-id="1665"
        ></path>
        <path
          d="M512 780m-36 0a36 36 0 1 0 72 0 36 36 0 1 0-72 0Z"
          fill="#1A1A1A"
          p-id="1666"
        ></path>
        <path
          d="M512 208c-103.83 0-188 84.17-188 188 0 1.42 0 2.84 0.05 4.25H324a36 36 0 0 0 72 0h0.09c-0.06-1.41-0.09-2.83-0.09-4.25a116 116 0 1 1 116 116 36 36 0 0 0-36 36v88a36 36 0 0 0 36 36 36 36 0 0 0 36-36v-55.46c86.61-16.8 152-93 152-184.54 0-103.83-84.17-188-188-188z"
          fill="#1A1A1A"
          p-id="1667"
        ></path>
      </svg>
    </div>
    <div class="card-content">{{ count }}</div>
    <div class="card-charts">
      <slot name='charts'></slot>
    </div>
    <div class="card-footer">
      <slot name='footer'></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: "Detail",
  props:['title','count']
};
</script>

<style scoped>
.card-header{
  display:flex;
  justify-content: space-between;
}
.card-content{
  font-size: 25px;
  padding:10px 0
}
.card-charts{
  height:50px ;
}
.card-footer{
  border-bottom: 1px solid black;
}
</style>

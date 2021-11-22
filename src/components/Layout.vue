<template>
  <div class="layout-wrapper" :class="classPrefix && `${classPrefix}-wrapper`">
    <div class="content" :class="classPrefix && `${classPrefix}-content`">
      <slot></slot>
    </div>
    <Nav></Nav>
  </div>
</template>

<script lang="ts">
  export default {
    props: ['classPrefix'],
    name: 'Layout'
  }
</script>

<style lang="scss" scoped>
.layout-wrapper{
  background-color: #f5f5f5;
  display: flex;
  flex-direction: column;
  height: 100vh;
}
.content{
  overflow: auto;
  flex-grow: 1;
}
</style>
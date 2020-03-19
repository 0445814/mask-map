<template>
  <div class="loading">
    <div class="pacman" style="width: 100px">
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
    </div>
    <slot></slot>
  </div>
</template>
<script>
import 'loaders.css';

export default {};
</script>
<style lang="scss">
// @import url('loader.css');
.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: #73c0d8;
  z-index: 99999;
}
</style>

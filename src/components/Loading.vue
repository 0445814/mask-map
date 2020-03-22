<template>
  <transition>
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
  </transition>
</template>
<script>
import 'loaders.css';

export default {};
</script>
<style lang="scss" scoped>
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
.v-leave {
  opacity: 1;
}
.v-leave-active {
  transition: opacity 1s;
}
.v-leave-to {
  opacity: 0;
}
.v-enter {
  opacity: 0;
}
.v-enter-active {
  transition: opacity 1s;
}
.v-enter-to {
  opacity: 1;
}
</style>

<script setup>
import OrbitNavigation from "@/components/OrbitNavigation.vue";
</script>

<template>
  <div id="app">
    <orbit-navigation
        :base-radius="245"
        :num-days-visible="10"
        :radius-increment="100"/>
  </div>
</template>

<style scoped>
#app {
  height: 100vh;
  width: 100%;
  background: black;
}
</style>

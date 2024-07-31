<template>
  <div>
    <Landingpage v-if="!activateMainpage" @leave="leaveLandingpage"/>
    <Mainpage v-if="activateMainpage" :activeImage="activeImage" @setActiveImage="setActiveImage"/>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
// import ThreejsScene from './components/ThreejsScene.vue'
import Mainpage from './components/Mainpage.vue'
import Landingpage from './components/Landingpage.vue'

const activeImage = ref(0)
const setActiveImage = (index: number) => {
  activeImage.value = index
}
const activateMainpage = ref(false)
const leaveLandingpage = () => {
  activateMainpage.value = true
}
</script>

<style scoped>

</style>

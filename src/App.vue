<template>
  <div>
    <Landingpage25 v-if="!activateMainpage" @leave="leaveLandingpage"/>
    <Mainpage v-if="activateMainpage" :activeImage="activeImage" @setActiveImage="setActiveImage"/>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
// import ThreejsScene from './components/ThreejsScene.vue'
import Mainpage from './components/Mainpage4.vue'
import Landingpage25 from './components/Landingpage25.vue'

const activeImage = ref(0)
const setActiveImage = (index: number) => {
  activeImage.value = index
}
const activateMainpage = ref(true)
const leaveLandingpage = () => {
  activateMainpage.value = true
}
</script>

<style scoped>

</style>

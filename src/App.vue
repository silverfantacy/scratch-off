<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import ScratchOff from './components/ScratchOff.vue'
import brushImage from './assets/brush.png'
import coverImage from './assets/cover.png'
</script>

<template>
  <div class="container">
    <ScratchOff :brush-image="brushImage" :cover-image="coverImage">
      <template #secret>
        <div style="display: flex;flex-direction: column;justify-content: center;align-items: center;">
          <img :src="brushImage" alt="">
          <h3>哭啊～沒中獎</h3>
        </div>
      </template>
    </ScratchOff>
  </div>
</template>

<style scoped>
.container {
  width: 100%;
  height: 100%;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>

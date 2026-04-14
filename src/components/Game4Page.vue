<script setup>
import CurrentSingleBallGamePage from './CurrentSingleBallGamePage.vue'
</script>

<template>
  <CurrentSingleBallGamePage
    v-bind="$attrs"
    :show-tema-side-plays="false"
    :show-tema-section="false"
    :show-ball-selector="false"
  />
</template>

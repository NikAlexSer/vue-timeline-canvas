<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
   
    <Timeline/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import Timeline from '@/components/Timeline.vue'; // @ is an alias to /src

export default Vue.extend({
  name: 'home',
  components: {
    Timeline,
  },
});
</script>

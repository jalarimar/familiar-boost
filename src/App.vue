<template>
  <v-app>
    <v-main class="dark-bg">
      <HelloWorld/>
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',
  components: {
    HelloWorld,
  },
};
</script>

<style scoped>
.dark-bg {
  background-color: #090909;
}
</style>
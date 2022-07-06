<script setup>
import TabsWrapper from "./components/TabsWrapper.vue";
import Tab from "./components/Tab.vue";
</script>

<template>
  <tabs-wrapper>
    <tab title="Tab 1"> Hello from Tab 1 </tab>
    <tab title="Tab 2"> Hello from Tab 2 </tab>
    <tab title="Tab 3"> Hello from Tab 3 </tab>
    <tab title="Tab 4"> Hello from Tab 4 </tab>
  </tabs-wrapper>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

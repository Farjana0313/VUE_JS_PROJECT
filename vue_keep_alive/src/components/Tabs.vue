<template>
  <div>
    <button v-for="tab in tabs" :key="tab" @click="component=tab">
      {{ tab }}
    </button>
    <component :is="component"/>
  </div>
</template>

<script>
import About from "./About.vue";
import Contact from "./Contact.vue";

export default {
  name: "Tabs",
  components: {About, Contact},
  data() {
    return {
      tabs: ["About", "Contact"],
      component: "About"
    }
  }
}
</script>

<style scoped>

</style>
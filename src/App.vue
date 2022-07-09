<template>
  <div class="container">
    <h2>Choose your skills</h2>
    <MultiSelect
      placeholder="Search your skills"
      v-model="value"
      :taggable="true"
      :options="options"
    />
  </div>
</template>

<script>
import MultiSelect from "./components/MultiSelect.vue";

export default {
  name: "App",
  data() {
    return {
      value: null,
      options: ["Vue.js", "React", "Angular", "Python", "UI/UX"],
    };
  },
  components: {
    MultiSelect,
  },
};
</script>

<style>
#app {
  font-family: Verdana, Geneva, Tahoma, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  min-height: 100vh;
}
.container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  min-height: 20vh;
}
</style>

<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Welcome to my Vue 3 project!</p>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      title: "Hello World Component",
    };
  },
};
</script>

<style scoped>
h1 {
  color: #42b983;
}
</style>

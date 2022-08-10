<template>
  <div id="app">
    <TokenSwap />
  </div>
</template>

<script>
import TokenSwap from './components/TokenSwap.vue'

export default {
  name: 'App',
  components: {
    TokenSwap
  }
}
</script>

<style>
  body {
    background-color: #fcfcfc
  }

  .container {
    background-color: #232939;
    width: 530px;
    margin: auto;
    padding: 10px;
    border-radius: 10px;
    box-shadow: #f8f8f8 2px 2px 3px;
  }

  .candidate-container {
    display: flex;
    flex-direction: row;
  }

  .candidate-card {
    background-color: #ffffff;
    width: auto;
    margin: 8px 8px;
    padding: 10px;
    border-radius: 10px;
    box-shadow: #5e5e5e 1px 1px 8px;
  }

  .candidate-count {
    color: #e9e9e9;
  }

  .title {
    color: #e9e9e9;
  }

  .error-badge {
    margin: 5px 0px;
    color: #ff4551;
  }

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Roboto&display=swap');
body {
  margin: 0;
  font-size: 2rem;
  font-family: 'Roboto';
}
.page {
  text-align: center;
  /* nesting for the need to test postcss */
  code {
    background-color: #f0f0f0;
    padding: 3px 5px;
    border-radius: 2px;
  }
}

</style>

<script>
//import * as d3 from 'd3';

export default {
  name: "App",
  data() {
    return {
      
    }
  }
};
</script>

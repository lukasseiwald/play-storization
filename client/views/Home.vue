<template>
  <div class="page">
    <Header></Header>
    <div class="pieWrapper">
      <PieChart></PieChart>
      <DonutChart></DonutChart>
    </div>
    <ScatterPlot></ScatterPlot>
    <Butterfly></Butterfly>
    <Footer></Footer>
  </div>
</template>

<script>
import Header from '../components/templates/Header'
import Footer from '../components/templates/Footer'
import PieChart from '../components/charts/PieChart'
import DonutChart from '../components/charts/DonutChart'
import ScatterPlot from '../components/charts/Scatter'
import Butterfly from '../components/charts/Butterfly'

import * as d3 from 'd3';

export default {
  components: {
    Header,
    PieChart,
    DonutChart,
    ScatterPlot,
    Butterfly,
    Footer
  },
  mounted() {
    // d3.json('/static/demo_data/quantifiable/populations.json').then(res => {
    // d3.csv('/data/googleplaystore.csv').then(res => {
    //   this.rawData = res
    //   console.log(this.rawData)
    // })
  },
}
</script>

<style lang="scss">
  .page {
    overflow: hidden;
  }
  .pieWrapper {
    width: 100%;
    height: 16em;
    margin: 3em 0em;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
  }
</style>

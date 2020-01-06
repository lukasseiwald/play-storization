<template>
  <div id='bar-chart' class='bar-chart-container'>
    <div class="half-circle-spinner" id="spinner2">
			<div class="circle circle-1"></div>
			<div class="circle circle-2"></div>
		</div>
    <div class='bar-chart-container__left app'>
      <div class='app__header'>
        <a href='./' target='_blank' class='app__name'></a>
        <img class='app__icon' src='' alt='' title=''>
        <p class="app__description">
        </p>
      </div>
      <div class="app__chart">
      </div>
    </div>
    <div class='bar-chart-container__right app'>
      <div class='app__header'>
        <img class='app__icon' src='' alt='' title=''>
        <a href='./' target='_blank' class='app__name'></a>
        <p class="app__description">
        </p>
      </div>
      <div class="app__chart">
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import { bus } from '../../app.js';
const gplay = require('google-play-scraper');

export default {
  name: 'Butterfly',
  data() {
    return {
      apps: new Array()
    }
  },
  methods: {
    fillWithData() {
      const dataArray = JSON.parse(JSON.stringify(this.apps));
      console.log(dataArray)
      this.fillContainerWithAppData('.bar-chart-container__left', dataArray[0]);
      this.createBarChartLeft('.bar-chart-container__left', dataArray[0]); //with class! for left and right
      if (dataArray[1]) {
        this.fillContainerWithAppData('.bar-chart-container__right', dataArray[1]);
        this.createBarChartLeft('.bar-chart-container__right', dataArray[1]);
      }
    },
    transformData(data) {
      let promises = new Array();
      for (let app of data) {
        promises.push(this.fetchDataFromStore(app.App));
      }
      return Promise.all(promises).then((values) => {
        let resultingApps = new Array();
        for (let i = 0; i < values.length; i++) {
          console.log(values[i])
          data[i].amountRatings = values[i].ratings;
          data[i].summary = values[i].summary;
          data[i].icon = values[i].icon;
          data[i].url = values[i].url;
          data[i].score = values[i].scoreText;
          data[i].descriptionLength = values[i].descriptionLength
        }
        return data;
      });
    },
    fetchDataFromStore(appName){
      console.log(appName)
      return new Promise(function(resolve, reject) {
         gplay.search({
          term: appName,
          num: 1
        })
        .then((res) => {
          gplay.app({appId: res[0].appId})
          .then((res) => {
            let app = {};
            app.amountRatings = res.ratings;
            app.summary = res.summary;
            app.icon = res.icon;
            app.url = res.url;
            app.score = res.scoreText;
            app.descriptionLength = res.description.length
            //header image
            console.log(res)
            resolve(app);
          });
        })
      });
    },
    createBarChartLeft(el, data) {
      let that = this;
      let element = document.querySelector(el + ' .bar-test');
      if (element) element.parentNode.removeChild(element);

      let right = false;
      if (el.includes('right')) {
        right = true;
      }
      console.log(right)

      let test = [{
        'Category': "Rating",
        'Value': data.Rating,
        'Max': 5
      },
      {
        'Category': 'Installs',
        'Value': data.Installs,
        'Max': 1000000000
      }
      ,
      {
        'Category': 'Reviews',
        'Value': data.Reviews,
        'Max': 60000000
      },
      {
        'Category': 'Description Length',
        'Value': data.descriptionLength,
        'Max': 4300
      },
      {
        'Category': 'Size in B',
        'Value': data.Size * 1000,
        'Max': 1000000000
      }
    ];

      if (!right) {
        var margin = {top: 20, right: 0, bottom: 30, left: 150},
            width = ((window.innerWidth/6)*2) - margin.left - margin.right,
            height = 500 - margin.top - margin.bottom;

        // set the ranges
        var y = d3.scaleBand()
                  .range([0, height])
                  .padding(0.1);
        var x = d3.scaleLinear()
                  .range([width, 0]);

        test.map(function(d){
            d.xScale  = d3.scaleLinear()
            .range([width, 0])
            .domain([0, d.Max]);
        });

        var svg = d3.select(el + " .app__chart").append("svg")
            .attr("class", "bar-test")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
          .append("g")
            .attr("transform",
                  "translate(" + margin.left + "," + margin.top + ")");

        y.domain(test.map(function(d) { return d.Category; }));

        svg.selectAll(".bar")
            .data(test)
          .enter().append("rect")
            .attr("class", "bar")
            .attr("y", function(d) { return y(d.Category); })
            .attr("height", y.bandwidth())
            .attr("x", function(d,i) { return width - (width - d.xScale(d.Value)); })
            .attr("width", function(d) { return width - d.xScale(d.Value); })
            .attr("fill", "#FFD400");

        svg.selectAll("text")
          .data(test)
          .enter()
          .append("text")
          .text(function(d) {
            if (d.Value == 0) {
              return 'Varies w/ device';
            }
            return that.fnum(d.Value);
          })
          .attr("text-anchor", "start")
          .attr("x", 7)
          .attr("y", function(d) { return (y(d.Category) + y.bandwidth()/2 + 15); })
          .attr("font-family", "sans-serif")
          .attr("font-size", "25px")
          .attr("fill", "black");

        svg.append("g")
            .call(d3.axisLeft(y));
      }
      else {

        var margin = {top: 20, right: 150, bottom: 30, left: 0},
            width = ((window.innerWidth/6)*2) - margin.left - margin.right,
            height = 500 - margin.top - margin.bottom;

        // set the ranges
        var y = d3.scaleBand()
                  .range([0, height])
                  .padding(0.1);
        var x = d3.scaleLinear()
                  .range([width, 0]);

        test.map(function(d){
            d.xScale  = d3.scaleLinear()
            .range([width, 0])
            .domain([0, d.Max]);
        });

        var svg = d3.select(el + " .app__chart").append("svg")
            .attr("class", "bar-test")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
          .append("g")
            .attr("transform",
                  "translate(" + margin.left + "," + margin.top + ")");

          y.domain(test.map(function(d) { return d.Category; }));

          svg.selectAll(".bar")
              .data(test)
            .enter().append("rect")
              .attr("class", "bar")
              .attr("y", function(d) { return y(d.Category); })
              .attr("height", y.bandwidth())
              .attr("x", 0)
              .attr("width", function(d) { return width - d.xScale(d.Value); })
              .attr("fill", "#3BCCFF");

          svg.selectAll("text")
            .data(test)
            .enter()
            .append("text")
            .text(function(d) {
              if (d.Value == 0) {
                return 'Varies w/ device';
              }
              return that.fnum(d.Value);
            })
            .attr("text-anchor", "end")
            .attr("x", width - 7 )
            .attr("y", function(d) { return (y(d.Category) + y.bandwidth()/2 + 15); })
            .attr("font-family", "sans-serif")
            .attr("font-size", "25px")
            .attr("fill", "black");
            // .attr("x", function(d) { return width - (width - x(d.Value)) + d.Value.toString().replace(/[^0-9]/g,"").length * 15 + 10; })

          // svg.append("g")
          //     .attr("transform", "translate(0," + height + ")")
          //     .call(d3.axisBottom(x));
          svg.append("g")
              .attr("transform", "translate(" + width + ",0)")
              .call(d3.axisRight(y));

        }
      d3.select('#spinner2').style('display', 'none');
    },
    fillContainerWithAppData(containerName, app) {
      let barChart = document.getElementById('bar-chart');
      let left = barChart.querySelector(containerName);
      let name = left.querySelector('.app__name');
      let icon = left.querySelector('.app__icon');
      let description = left.querySelector('.app__description');
      name.innerHTML = app.App
      icon.src = app.icon;
      name.href = app.url;
      description.innerHTML = app.summary;
    },
    fnum(x) {
      if(isNaN(x)) return x;
      if(x < 9999) {
        return x;
      }
      if(x < 1000000) {
        return Math.round(x/1000) + "K";
      }
      if( x < 10000000) {
        return (x/1000000).toFixed(2) + "M";
      }
      if(x < 1000000000) {
        return Math.round((x/1000000)) + "M";
      }
      if(x < 1000000000000) {
        return Math.round((x/1000000000)) + "B";
      }
      return "1T+";
    }
  },
  mounted() {
    let that = this;
    bus.$on('selectedDotInScatter', (selectedDots) => {
      d3.select('#spinner2').style('display', 'block');
      let selectedApps = selectedDots.map((el) => el.app)
      that.transformData(selectedApps).then(res => {
        that.apps = res;
        that.fillWithData();
      })
    })
    // bus.$on('deselectedDotInScatter', (app) => {
    //   let data = JSON.parse(JSON.stringify(that.apps))
    // })
  },
};
</script>


<style lang='scss'>
  * {
    box-sizing: border-box;
  }
  .bar-chart-container {
    margin-top: 2em;
    padding-top: 3.5em;
    padding-bottom: 2em;
    border-top: 1px solid black;
    display: flex;
    //background-color: green;
    flex-wrap: wrap;
    position: relative;

    .app__icon {
      max-width: 60px;
      max-height: 60px;
      border-radius: 14px;
    }
    .app__name {
      text-transform: uppercase;
      font-size: .8em;
      margin: 0;
      text-align: left;
      width: calc(50% - 80px);
      text-decoration: none;
      color: black;
      font-weight: 700;
      padding-left: 1vw;
      padding-right: 1vw;
      flex: 1;
      &:hover {
        //color: #3BCCFF;
        text-decoration: underline;
        cursor: pointer;
      }
    }
  }
  .app__header {
    display: flex;
    align-items: center;
    padding: 0 0.5em;
    flex-wrap: wrap;
  }
  .bar-chart-container__left,
  .bar-chart-container__right {
    display: flex;
    flex-wrap: wrap;
    width: 50vw;
    text-align: left;
  }
  .bar-chart-container__left {
    justify-content: flex-end;
      .app__header {
        display: flex;
        justify-content: flex-end;
        padding-left: 2em;
      }
      .app__name {
        text-align: right;
      }
      .app__description {
        text-align: right;
      }
  }
  .bar-chart-container__right {
    .app__header {
      padding-right: 2em;
    }
  }
  p {
    font-size: 0.7em;
  }
  .bar-chart-container__svg {
    width: 100%;
    .test-text {
      fill: #2C3E50;
      font-family: 'Roboto';
    }
  }
  .app__description {
    margin-top: 1em;
    width: 50vw;
    font-size: .5em;
  }
  .app__chart {
    margin-top: .5em;
    align-self: flex-end;
  }
  #spinner2 {
    display: none;
    top: 1em;
    margin: auto;
    right: auto;
    left: calc(50% - 30px);
  }
</style>

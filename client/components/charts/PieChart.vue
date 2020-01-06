<template>
  <div id="PieChart">
    <p id="instructionPie"> Select a Category to get a more detailled look </p>
		<div class="half-circle-spinner" id="spinner1">
			<div class="circle circle-1"></div>
			<div class="circle circle-2"></div>
		</div>
		<svg id="pie-chart"></svg>
		<div id="extraInfo"></div>
		<div id="backButton"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import { bus } from '../../app.js';


export default {
	name: "PieChart",
  data() {
    return {
			categories: new Array(),
			otherCategories: new Array(),
    }
  },
  props: {
	  selectedCategory: String
  },
  methods: {
    fetchData() {
      let that = this;
      //let arr = [];
			let categoryDict = {};

      d3.csv('../../../data/googleplaystore-transformed.csv').then(res => {
        for (let i = 0; i < res.length; i++) {
					if(!categoryDict[res[i].Category]) {
						categoryDict[res[i].Category] = 1;
					}
					else {
						categoryDict[res[i].Category] = categoryDict[res[i].Category] + 1;
					}
				}

				let sortedArr = Object.keys(categoryDict).map(function(key) {
					return [key, categoryDict[key]];
				});

				// Sort the array based on the second element
				sortedArr.sort(function(first, second) {
					return second[1] - first[1];
				});

				//that.count_categories = categories;
				let categoryArr = [];
				let index = 0;

				sortedArr.forEach(category => {
					categoryArr[index] = {
						name: category[0].replace(/_/g, " "),
						value: category[1]
					}
					index++;
				});

				that.categories = categoryArr.slice(0, 15);
				that.categories[that.categories.length] = {
					name: "Other",
					value: 1400 //fits well
				}
				that.otherCategories = categoryArr.slice(15, categoryArr.length)

				this.drawPie(that.categories, false);
      })
		},
		drawPie(categories, showOthers) {
			let that = this;
			// Define size & radius of pie chart
			var width = 500,
				height = 500,
				radius = Math.min(width, height) / 2.1;

			d3.select('#spinner1').style('display', 'none');
			bus.$emit('chosenGenre', "ALL", "ALL", "");

			// Define arc colours
			let colors = ['#003f5c', '#374c80', '#7a5195', '#bc5090', '#ef5675', '#ff764a', '#ffa600', '#e6194b', '#3cb44b', '#4363d8', '#30BFBF', '#4d850e', '#FFC154', '#F47A1F', '#E8A09A', '#9BBFE0', '#64C2A6', '#808000', '#000075', '#808080', '#000000', '#bc5090', '#ef5675'];
			let	colorCounter = 0;

			d3.select("#instructionPie").style("opacity", "1");
			d3.select("#instructionDonut").style("opacity", "0");

			if(showOthers) {
				var backButton = d3.select("#backButton")
					.style("display", "block")
					.text("SHOW ALL")
					.on("click", function() {
						d3.select('#mainPie').remove();
						that.drawPie(that.categories, false);
					})
			}
			else {
				d3.select("#backButton").style("display", "none")
			}

			// Define arc ranges
			var arcText = d3.scaleOrdinal()
				.range([0, width]);

			// Determine size of arcs
			var arc = d3.arc()
				.innerRadius(radius - 237)
				.outerRadius(radius);

			var arcOver = d3.arc()
				.innerRadius(radius - 237)
				.outerRadius(radius + 8);

			// Create the pie chart layout
			var pie = d3.pie()
				.value(function (d) { return d["value"]; })
				.sort(null)

			var extraInfo = d3.select("#extraInfo");

			// Append SVG attributes and append g to the SVG
			var svg = d3.select("#pie-chart")
				.attr("width", width)
				.attr("height", height)
				.append("g")
				.attr("id", "mainPie")
				.attr("transform", "translate("+ 248 + "," + 246 + ")");

			// Calculate SVG paths and fill in the colours
			var g = svg.selectAll(".arc")
				.data(pie(categories))
				.enter().append("g")
				.attr("class", "arc")

				// Make each arc clickable
				.on("click", function(d, i) {
					if(categories[i].name == "Other") {
						d3.select('#mainPie').remove();
						that.drawPie(that.otherCategories, true);
					}
					else {
						bus.$emit('childToParent', categories[i].name, colors[i]);
						d3.select('#mainPie').remove();
						d3.select('#extraInfo').style('opacity', '0');
					}
				})

				.on('mouseover', function (d, i) {
					extraInfo.transition().duration('50')
						.style("opacity", 1)
						.style("color", "grey")
					let num = categories[i].name + '\n'+ (Math.round((categories[i].value / 9659)* 100)).toString() + '%';
					extraInfo.html(num)
				})
				.on('mouseout', function (d, i) {
					//Makes the new div disappear:
					extraInfo.transition()
						.duration('50')
						.style("opacity", 0);
				})

			// Append the path to each g
			var p = g.append("path")
				.attr("d", arc)
				.style('opacity', '1')
				.attr("fill", function(d, i) {
					return colors[colorCounter++];
				})

				p.on('mouseover', function (d, i) {
					d3.select(this).transition()
						.duration('100')
						.attr("id", "piePath" + i)
						.attr("d", arcOver)
						.attr('fill-opacity', 0.8);
				})
				.on('mouseout', function (d, i) {
					d3.select(this).transition()
						.duration('100')
						.attr("d", arc)
						.attr('fill-opacity', 1);
				})
				.transition().delay(function(d, i) { return i * 30; }).duration(30)

				

			// Append text labels to each arc
			g.append("text")
				.attr("transform", function(d) {
					var rotation = d.endAngle < Math.PI ? (d.startAngle / 2 + d.endAngle / 2) * 178 / Math.PI : (d.startAngle / 2 + d.endAngle / 2 + Math.PI) * 182 / Math.PI;
					return "translate(" + arc.centroid(d) + ") rotate(-97) rotate(" + rotation + ")";
				})
				.on('mouseover', function (d, i) {
					d3.select('#piePath' + i).transition()
						.attr("d", arcOver)
				})
				.on('mouseout', function (d, i) {
					d3.select('#piePath' + i).transition()
						.attr("d", arc)
				})
				.transition().delay(function(d, i) { return i * 40; }).duration(40)
				.attr("text-anchor", "middle").attr("alignment-baseline", "middle")
				.attr("dy", 4)
				.attr("dx", function(d) {
					if(d.endAngle < Math.PI) {
						return 40
					}
					return -40
				})
				.attr("fill", "#fff")
				.text(function(d,i) { return categories[i].name; })

			g.selectAll(".path text").call(that.wrap, arcText.range([0, width]));
		},
    // onClick(item, index) {
		//   //to see further Genres, emit to Donut Chart
		// 	this.props.selectedCategory = item;
		// 	bus.$emit('childToParent', item);
		// 	svg.style('display', 'none')
		// 	d3.select("#backButton").remove()
		// 	d3.select("#mainPie").style('z-index', -2)
		// }
  },
  mounted() {
		this.fetchData();
		bus.$on('goBack', () => {
			this.drawPie(this.categories, false);
    })
	}
}
</script>


<style lang="scss">
  #PieWrapper {
    position: relative;
  }
	#PieChart {
		position: absolute;
	}
  p {
    font-size: 0.7em;
  }
  svg {
    .test-text {
      fill: #2C3E50;
      font-family: 'Roboto';
    }
  }
	svg#pie-chart {
		display: block;
		margin: 0 auto;
	}

	text.inner-circle {
		font-weight: 400;
		font-size: 12px;
		text-transform: uppercase;
	}

	.arc {
		cursor: pointer;

		text {
			font-weight: 300;
			font-size: 12px;
			color: #fff;
		}
	}

	#extraInfo {
		position: absolute;
    top: 60%;
    transform: translateY(-50%);
		opacity: 0;
		width: 4.8em;
		height: 5em;
		text-align: left;
		font-size: 0.7em;
    right: -25%;
	}

	#backButton {
		position: absolute;
		display: none;
		width: 5em;
		text-align: center;
		font-size: 0.4em;
		top: 7em;
		-ms-flex-align: center;
		padding: 1em 1em;
		cursor: pointer;
		color: black;
		border-radius: 3em;
		background-color: #3BCCFF;
	}

	#instructionPie {
		opacity: 0;
	}

	.half-circle-spinner, .half-circle-spinner * {
      box-sizing: border-box;
    }

    .half-circle-spinner {
      width: 60px;
      height: 60px;
      border-radius: 100%;
      position: absolute;
			top: 3em;
			left: 0;
      margin: 0 auto;
      right: 0;
    }

    .half-circle-spinner .circle {
      content: "";
      position: absolute;
      width: 100%;
      height: 100%;
      border-radius: 100%;
      border: calc(60px / 10) solid transparent;
    }

    .half-circle-spinner .circle.circle-1 {
      border-top-color: #3BCCFF;
      animation: half-circle-spinner-animation 1s infinite;
    }

    .half-circle-spinner .circle.circle-2 {
      border-bottom-color: #3BCCFF;
      animation: half-circle-spinner-animation 1s infinite alternate;
    }

    @keyframes half-circle-spinner-animation {
      0% {
        transform: rotate(0deg);

      }
      100%{
        transform: rotate(360deg);
      }
    }
</style>

<template>
  <div id="DonutChart">
	<p id="instructionDonut">You can also select a genre</p>
		<svg id="donut-chart"></svg>
		<div id="backButton2"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import { bus } from '../../app.js';

export default {
  name: "DonutChart",
  data() {
    return {
			genres: new Array(),
			category: 'GAME'
    }
  },
  methods: {
    fetchData(filtered_category, singleColor) {
      let that = this;
      //let arr = [];
	  let genreDict = {};
	  
	  bus.$emit('chosenGenre', filtered_category, filtered_category, singleColor);

      d3.csv('../../../data/googleplaystore-transformed.csv').then(res => {
        for (let i = 0; i < res.length; i++) {
					if(res[i].Category === filtered_category) {
						if(!genreDict[res[i].Genres]) {
							genreDict[res[i].Genres] = 1;
						}
						else {
							genreDict[res[i].Genres] = genreDict[res[i].Genres] + 1;
						}
					}
				}
				
				let sortedArr = Object.keys(genreDict).map(function(key) {
					return [key, genreDict[key]];
			});

		// Sort the array based on the second element
		sortedArr.sort(function(first, second) {
			return second[1] - first[1];
		});

		//that.count_genres = genres;
		let categoryArr = [];
		let index = 0;

		sortedArr.forEach(category => {
			categoryArr[index] = {
				name: category[0],
				value: category[1]
			}
			index++;
		});

		that.genres = categoryArr.slice(0, 8);

		if(that.genres.length < 2) {
			that.genres[0] = {
				name: filtered_category,
				value: 1
			}
		}
		this.drawPie(filtered_category, singleColor);	
    })
		},
		drawPie(filtered_category, singleColor) {
			let that = this;
			// Define size & radius of donut pie chart
			var width = 500,
				height = 500,
				radius = Math.min(width, height) / 2;

			// Define arc colours
			let colors = [singleColor, '#7a5195', '#bc5090', '#ef5675', '#ff764a', '#ffa600', '#e6194b', '#3cb44b', '#4363d8', '#911eb4', '#9a6324', '#800000', '#808000', '#000075', '#808080', '#000000', '#bc5090', '#ef5675'];
			let	colorCounter = 0;

			d3.select("#instructionDonut").style("opacity", "1");
			d3.select("#instructionPie").style("opacity", "0");

			// Define arc ranges
			var arcText = d3.scaleOrdinal()
				.range([0, width]);

			// Determine size of arcs
			var arc = d3.arc()
			.innerRadius(radius - 120)
			.outerRadius(radius - 14);

			var arcOver = d3.arc()
      .innerRadius(radius - 128)
      .outerRadius(radius - 6);

			// Create the donut pie chart layout
			var pie = d3.pie()
				.value(function (d) { return d["value"]; })
				.sort(null)

			var backButton = d3.select("#backButton2")
				.text("GO BACK")
				.style("display", "block")
				.on("click", function() {
					d3.select('#mainDonut').remove();
					d3.select('#backButton2').style('display', 'none');
					d3.select("#DonutChart").style('z-index', '-2')
					bus.$emit('goBack');
					that.singleColor = null;
				})

			// Append SVG attributes and append g to the SVG
			var svg = d3.select("#donut-chart")
				.attr("width", width)
				.attr("height", height)
				.append("g")
				.attr("id", "mainDonut")
				.attr("transform", "translate(" + radius + "," + radius + ")");

			d3.select("#DonutChart").style('z-index', '20')
			d3.select('#backButton').style('display', 'none');

			// Calculate SVG paths and fill in the colours
			var g = svg.selectAll(".arc")
				.data(pie(this.genres))
				.enter().append("g")
				.attr("class", "arc")

				// Make each arc clickable 
				.on("click", function(d, i) {
					bus.$emit('chosenGenre', filtered_category, that.genres[i].name, colors[0]);
					var container = d3.select("#scatterPlot");
      				container.scrollTop = container.scrollHeight;
					d3.select('#scatterPlot').remove();
				})
			
			document.getElementsByTagName("g")[0].setAttribute("id", "democlass");

			// Define inner circle
			svg.append("circle")
			.attr("cx", 0)
			.attr("cy", 0)
			.attr("r", 100)
			.attr("fill", "#35a3ca") ;

			// Append the path to each g
			g.append("path")
				.attr("d", arc)
				.on('mouseover', function (d, i) {
					d3.select(this).transition()
						.duration('100')
						.attr("id", "path" + i)
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
				.style('opacity', '1')
				.attr("fill", function(d, i) {
					return colors[colorCounter++];
				})

					// Append text labels to each arc
			g.append("text")
				.attr("transform", function(d) {
					var rotation = d.endAngle < Math.PI ? (d.startAngle / 2 + d.endAngle / 2) * 178 / Math.PI : (d.startAngle / 2 + d.endAngle / 2 + Math.PI) * 182 / Math.PI;
					return "translate(" + arc.centroid(d) + ") rotate(-97) rotate(" + rotation + ")";
				})
				.on('mouseover', function (d, i) {
					d3.select('#path' + i).transition()
						.attr("d", arcOver)
				})
				.on('mouseout', function (d, i) {
					d3.select('#path' + i).transition()
						.attr("d", arc)
				})
				.attr("dy", ".35em")
				.style("text-anchor", "middle")
				.attr("fill", "#fff")
				.text(function(d,i) { 
					if(that.genres[i].name.includes(';')) {
						return that.genres[i].name.split(';')[1]
					}
					return that.genres[i].name; 
				})
					
					// Append text to the inner circle
			svg.append("text")
				.attr("dy", "0.5em")
				.style("text-anchor", "middle")
				.attr("class", "inner-circle")
				.attr("fill", "#ffffff")
				.text(function(d) { return that.category; });  
				
			g.selectAll(".arc text").call(that.wrap, arcText.range([0, width]));
		}
  },
  mounted() {
		bus.$on('childToParent', (data, color) => {	
			this.category = data;
			this.fetchData(data, color);
		});
  },
};
</script>


<style lang="scss">
	#DonutChart {
		z-index: -1;
		margin-right: 0.1em;
		margin-bottom: 0.1em;
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
	svg#donut-chart {
		display: block;
		margin: 0 auto;
	}

	text.inner-circle {
		font-weight: 400;
		font-size: 16px;
		text-transform: uppercase;
		padding-bottom: 0.2em;
	}

	.arc {
		cursor: pointer;
		
		text {
			font-weight: 300;
			font-size: 12px;
			color: #fff;
		}
	}

	#instructionDonut {
		opacity: 0;
	}

	#backButton2 {
		display: none;
		position: absolute;
		width: 5em;
		text-align: center;
		font-size: 0.4em;
		top: 4.5em;
		margin-left: 0.1em;
		-ms-flex-align: center;
		padding: 1em 1em;
		cursor: pointer;
		color: black;
		border-radius: 3em;
		background-color: #3BCCFF;
	}
</style>

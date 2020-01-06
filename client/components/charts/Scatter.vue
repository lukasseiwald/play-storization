<template>
  <div id="scatterplot">
	<p>Selected Category: <span id="selectedCategory" style="color: #3BCCFF; font-weight: bold">ALL</span></p>
    <p>Select two apps to compare them in detail!</p>
		<div id="scatterplot__hover-info">
		</div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import { bus } from '../../app.js';

export default {
  name: "ScatterPlot",
  components: {
  },
  data() {
    return {
		apps: new Array()
    }
  },
  methods: {
    fetchData() {
      let that = this;
      return d3.csv('../../../data/googleplaystore-transformed.csv').then(res => {
          res.forEach(function(d) {
            d.Rating = parseInt(d.Rating);
            d.Reviews = parseInt(d.Reviews);
            d.Installs = parseInt(d.Installs);
        });
        that.apps = res;
      })
    },
    getGenre(genre, isGenre) {
      let that = this;
			let appsArr = new Array();
      let res = JSON.parse(JSON.stringify(this.apps));

      if(!isGenre) {
        genre = genre.split(' ').join('_'); //BOOKS AND REFERENCES -> BOOKS_AND_REFERENCES
      }

      for (let i = 0; i < res.length; i++) {
				if(res[i].Genres == genre) {
					appsArr.push(res[i]);
				}
				else if(res[i].Category == genre) { //get Apps that do not have a genre, just category
					appsArr.push(res[i]);
				}
			}
      return appsArr;
		},
		drawScatter(data) {
      let el = document.getElementById('scatterplot__plot');
      if (el) el.parentNode.removeChild(el)

      let selectedDots = new Array();

			var margin = {top: 20, right: 20, bottom: 40, left: 40},
				width = 960 - margin.left - margin.right,
				height = 500 - margin.top - margin.bottom;

      var panExtent = {x: [0,width], y: [0,height] };

			// set the ranges
			var x = d3.scaleLinear()
      .range([0, width])
      .domain([0, d3.max(data, function(d) { return d.Reviews; })]).nice();

      var y = d3.scaleLinear()
      .range([height, 0])
      .domain([0, d3.max(data, function(d) { return d.Installs; })]).nice();

			var svg = d3.select("#scatterplot").append("svg")
				.attr("id", "scatterplot__plot")
				.attr("width", width + margin.left + margin.right)
				.attr("height", height + margin.top + margin.bottom)
				.append("g")
				.attr("transform", "translate(40,20)");

        var zoom = d3.zoom()
            .scaleExtent([1, 50])
            .extent([[0, 0], [width, height]])
            .on("zoom", zoomed);

        let zoomer = svg.append("rect")
            .attr("width", width)
            .attr("height", height)
            .style("fill", "none")
            .style("pointer-events", "all")
            .call(zoom);

        //make grid in back

        let axisX2 = d3.axisBottom(x).tickFormat(d3.format('.2s'));
        let axisY2 = d3.axisLeft(y).tickFormat(d3.format('.2s'));

        let gX2 = svg.insert("g")
          .attr("class", "grid grid-x")
          .attr("transform", "translate(0," + height + ")")
          .call(axisX2
            .tickSize(-height)
            .tickFormat(''));

        let gY2 = svg.insert("g", '#scatterplot')
          .attr("class", "grid grid-y")
          .call(axisY2
            .tickSize(-width)
            .tickFormat(''));

        svg.selectAll('.grid')
          .selectAll('line')
          .attr('stroke', 'lightgray')
          .attr('opacity', '0.5');

			//Add the scatterplot
			let points = svg.selectAll("dot")
					.data(data)
				  .enter().append("circle")
					.attr("r", function(d) { return (d.Rating * 3); })
					.attr("cx", function(d) { return x(d.Reviews) })
					.attr("cy", function(d) { return y(d.Installs); })
					.style("cursor", "pointer")
					.style("fill", "#3BCCFF")
          .attr("fill-opacity","0.5")
					.on("mouseover", function(d) {
						var mouse = d3.mouse(this);
						d3.select('#scatterplot__hover-info').transition()
								.duration(0)
								.style("opacity", .9)
								.attr("left", mouse[0] +'px')
        				.attr("top", mouse[1] +'px')
						d3.select('#scatterplot__hover-info').html("<b>" + d.App + "</b><br/>Reviews: " + d.Reviews
						+ "<br/>Installs: " + d.Installs + "<br/>Rating: " + d.Rating)
					})
					.on("mouseout", function(d) {
								d3.select('#scatterplot__hover-info').transition()
									.duration(10)
									.style("opacity", 0);
					})
          .on("click", function(d) {
            if (selectedDots.length > 1) {
              let dot = selectedDots[0].dot
              let app = selectedDots[0].app;
              d3.select(dot)
                .style("fill", "#3BCCFF")
                .attr("fill-opacity","0.5")
              let removedDot = selectedDots.shift();
              bus.$emit('deselectedDotInScatter', removedDot.app);
            }
            selectedDots.push({dot: this, app: d});
            bus.$emit('selectedDotInScatter', selectedDots);
            d3.select(this)
              .style("fill", "#FFD400")
              .attr("fill-opacity","1")
          });

          svg.append("rect")
              .attr("width", margin.left)
              .attr("height", height + margin.top + margin.bottom)
              .attr("x", -margin.left)
              .attr("y", -margin.top)
              .style("fill", "white")

          svg.append("rect")
              .attr("width", width + margin.bottom + margin.top)
              .attr("height",margin.bottom)
              .attr("x", 0)
              .attr("y", height)
              .style("fill", "white")


			  // x-axis

        let axisX = d3.axisBottom(x).tickFormat(d3.format('.2s'));

				let gX = svg.append("g")
						.attr("class", "x axis")
						.attr("transform", "translate(0," + height + ")")
				    .call(axisX);

        gX.append("text")
						.attr("class", "label")
						.attr("x", width)
						.attr("y", -6)
						.style("text-anchor", "end")
						.attr("font-size", "8")
						.text("Reviews")
            .style('fill', 'black')

				// y-axis
        let axisY = d3.axisLeft(y).tickFormat(d3.format('.2s'));

				let gY = svg.append("g")
						.attr("class", "y axis")
						.call(axisY);

        gY.append("text")
						.attr("class", "label")
						.attr("transform", "rotate(-90)")
						.attr("y", 6)
						.attr("dy", ".71em")
						.style("text-anchor", "end")
						.text("Installs")
            .style('fill', 'black')

        function zoomed() {
          let transform = d3.event.transform;
          let tx = transform.x
          let ty = transform.y

           if (tx > 0) {
             tx = 0;
             transform.x = tx
           }
           if (ty < (height * (1-transform.k)) ) {
             ty = height * (1-transform.k);
             transform.y = ty
           }

        // create new scale ojects based on event
            var new_xScale = transform.rescaleX(x);
            var new_yScale = transform.rescaleY(y);
        // update axes
            gX.call(axisX.scale(new_xScale));
            gY.call(axisY.scale(new_yScale));

            gY2.call(axisY2.scale(new_yScale)
                .tickSize(-width)
                .tickFormat(''));
            gX2.call(axisX2.scale(new_xScale));

            svg.selectAll('.grid')
              .selectAll('line')
              .attr('stroke', 'lightgray');

            points.data(data)
             .attr('cx', function(d) {return new_xScale(d.Reviews)})
             .attr('cy', function(d) {return new_yScale(d.Installs)});

         }

    }
  },
  mounted() {
    this.fetchData().then( () => {
    	this.data = JSON.parse(JSON.stringify(this.apps))
      	this.drawScatter(this.data);
    })
		bus.$on('chosenGenre', (category, genre, color) => {
			if(genre != "ALL") {
        let apps
        if(category != genre) {
          apps = this.getGenre(genre, true)
        }
        else {
			  	apps = this.getGenre(genre, false)
        }
				this.drawScatter(apps);
				d3.select('#selectedCategory').style("color", color)
			}
			else {
        this.drawScatter(this.data);
        d3.select('#selectedCategory').style("color", "#3BCCFF")
      }
      if(category == genre) {
        d3.select('#selectedCategory').html(`${category}`)
      }
      else {
         if(genre.includes(';')) {
          //BOOKS AND REFERENCES -> BOOKS_AND_REFERENCES
          d3.select('#selectedCategory').html(`${category} (${genre.split(';')[1]})`)
        }
        else {
          d3.select('#selectedCategory').html(`${category} (${genre})`)
        }
      }
    })
	}
}
</script>


<style lang="scss">
  #scatterplot {
    position: relative;
  }
  #scatterplot__hover-info {
    padding: .5em;
		opacity: 0;
		position: absolute;
		right: 0;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    max-width: 250px;
    margin: 0 auto;
		font-size: 0.6em;
		text-align: left;
		border: 0.1em solid black;
    pointer-events: none;
	}

  .grid .domain {
    opacity: 0;
  }
</style>

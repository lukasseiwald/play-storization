<template>
  <div id="chart1">
    <!-- <p>Counts of Ratings 1-5</p> -->
    <svg viewBox="0 50 200 170">
      <rect
        v-for="(item, index) in abs_ratings"
        :key="index"
        :x="index * 5"
        :y="200 - item[0] / 10"
        :height="item[0] / 10"
        ref="rects"
        width="4.7"
        fill="#FFD400"
        @click="onClick(item, index)"
      />
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: "Chart1",
  data() {
    return {
      rating: [],
      abs_ratings: []
    }
  },
  methods: {
    fetchData() {
      let that = this;
      let arr = [];
      let currentRating = 0;

      d3.csv('../../../data/googleplaystore-transformed').then(res => {
        for (var i = 0; i < res.length; i++) {
          currentRating = Number(res[i].Rating)
          currentRating = Number.isNaN(currentRating) ? 0 : currentRating;

          that.rating.push(
            currentRating
          );
        }
        that.abs_ratings = this.getAbsRating(that.rating);
        console.log(that.abs_ratings);
      })
    },
    getAbsRating(arr) {
      //console.log(this.rating);
      let a = [], b = [], prev;

      arr.sort();
      for (let i = 0; i < arr.length; i++ ) {
          if ( arr[i] !== prev ) {
              a.push(arr[i]);
              b.push(1);
          } else {
              b[b.length-1]++;
          }
          prev = arr[i];
      }

      let c = [];

      for(let j = 1; j < a.length; j++) {
          c[j - 1] = [b[j], a[j]];
      }

      return c;
    },
    onClick(item, index) {
      console.log(item, index);
    }
  },
  mounted() {
    this.fetchData();
    //this.getAbsRating();
  },
};
</script>


<style lang="scss">
  p {
    font-size: 0.7em;
  }
  svg {
    .test-text {
      fill: #2C3E50;
      font-family: 'Roboto';
    }
  }
</style>

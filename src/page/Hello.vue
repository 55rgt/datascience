<template lang="pug">
  .wrapper
    .nav-container
      .topic-container
        .topic-title Korean Petition Insights
      .setting-container.section-container
        .desc-container
          .desc.setting-desc settings
        .setting-option-container
          .setting-option.setting-title Number of Trials
          .setting-input-wrapper
            input(v-model='numOfTrials' type="text").setting-input
          .setting-option(@click='clearGraph()').setting-button Clear
        .setting-option-container
          .setting-option.setting-title Render Time (Sec)
          .setting-input-wrapper
            input(v-model='renderTime' type="text").setting-input
          .setting-option(@click='passData()').setting-button Start
      .category-container.section-container
        .desc-container
          .desc charts
          .number {{ cCList.length }}
          i.icons.material-icons(@click='resetCategory()') refresh
        .category-list-container
          category_components(v-for="c in cCList" v-bind="c" v-on:toggleCategory="toggleCategory")
    .main-container
      .topic-container
      .display-container
        prediction-component(v-for="p in predictionList" v-bind="p" :ref="p.refName")
</template>

<script>

import Vue from 'vue';
import _ from 'lodash';

import category from '../../data/category.json';
import navCategory from '../../data/nav_category.json';
import { store } from '../vuex/store';

import predictionComponent from '../components/PredictionComponent';
import categoryComponent from '../components/Category-Component';

Vue.component(predictionComponent.name, predictionComponent);
Vue.component(categoryComponent.name, categoryComponent);

export default {
  name: 'HelloWorld',
  store,
  data() {
    return {
      predictionList: [],
      cCList: [],
      numOfTrials: 100,
      renderTime: 0.25,
      refreshIntervalID: null,
      numOfGrids: 7,
      count: 0
    };
  },
  created() {
    _.forEach(navCategory, (ele, i) => {
      this.cCList.push({
        category_title: category[ele.category].title,
        category_color: category[ele.category].color,
        category_category: ele.category,
        category_state: i === 0
      });
    });
    for (let i = 0; i < this.numOfGrids; i += 1) {
      this.predictionList.push({
        refName: `prediction_${i}`
      });
    }
  },
  methods: {
    passData() {
      this.clearGraph();
      this.refreshIntervalID = setInterval(this.renderGraph, this.renderTime * 1000);
    },
    renderGraph() {
      this.count += 1;
      if (this.count >= this.numOfTrials - 1) clearInterval(this.refreshIntervalID);
      for (let i = 0; i < this.numOfGrids; i += 1) {
        this.$refs[`prediction_${i}`][0].startRender(this.renderTime * 1000, this.count);
      }
    },
    clearGraph() {
      this.count = 0;
      clearInterval(this.refreshIntervalID);
      const that = this;
      const state = [];
      _.forEach(that.cCList, (ele) => {
        state[ele.category_category] = ele.category_state;
      });
      for (let i = 0; i < this.numOfGrids; i += 1) {
        this.$refs[`prediction_${i}`][0].initialize(state);
      }
    },
    toggleCategory(c) {
      if (c === '0') return;
      const selected = this.cCList.find(ele => ele.category_category === c);
      selected.category_state = !selected.category_state;
      const index = selected.category_category;
      const state = selected.category_state;
      for (let i = 0; i < this.numOfGrids; i += 1) {
        this.$refs[`prediction_${i}`][0].toggleChart(index, state);
        this.$refs[`prediction_${i}`][0].toggleInfoElement(index, state);
      }
    },
    resetCategory() {
      _.forEach(this.cCList, (ele, idx) => {
        if (idx === 0) ele.category_state = true;
        else ele.category_state = false;
        const index = ele.category_category;
        if (index !== '0') {
          const state = ele.category_state;
          for (let i = 0; i < this.numOfGrids; i += 1) {
            this.$refs[`prediction_${i}`][0].toggleChart(index, state);
            this.$refs[`prediction_${i}`][0].resetInfoElement();
          }
        }
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="sass">
@import url('https://fonts.googleapis.com/css?family=Playfair+Display:400,700')
@import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro:300')
.wrapper
  width: 1920px
  height: 1080px
  display: flex
  padding: 36px
  .nav-container
    width: 360px
    height: 100%
    .topic-container
      width: 100%
      height: 48px
      .topic-title
        width: 100%
        height: 48px
        font-size: 31px
        font-weight: bold
        font-style: normal
        font-stretch: normal
        letter-spacing: normal
        text-align: center
        color: #000000
        line-height: 48px
        font-family: 'Playfair Display', serif
    .setting-container
      width: 100%
      height: 150px
      .setting-option-container
        width: 100%
        height: 48px
        display: flex
        .setting-option
          width: 160px
          height: 100%
          line-height: 48px
          text-align: left
          font-size: 16px
        .setting-title
          padding-left: 6px
        .setting-input-wrapper
          width: 60px
          height: 100%
          display: flex
          align-items: center
          margin-right: 20px
          .setting-input
            width: 60px
            height: 36px
            text-align: center
            outline: none
            border-radius: 16px
            border: 1px solid silver
        .setting-button
          width: 80px
          text-align: center !important
          cursor: pointer
          border-radius: 16px
          &:hover
            background: rgba(0, 0, 0, 0.1)
    .category-container
      width: 100%
      height: 250px
      .category-list-container
        width: 100%
        height: 190px
        padding: 6px
        display: flex
        flex-wrap: wrap
        margin-top: 6px
    .rank-container
      width: 100%
      height: 512px
  .main-container
    width: calc(100% - 360px)
    height: 100%
    .topic-container
      width: 100%
      height: 48px
    .display-container
      width: 100%
      height: calc(100% - 48px)
      display: inline-flex
      flex-wrap: wrap
      padding: 12px

.section-container
  margin: 12px 0
  border-radius: 12px
  box-shadow: 3px 3px 3px 2px rgb(213, 222, 229)
  padding: 12px

.desc-container
  width: 100%
  height: 36px
  padding: 0 6px
  display: flex
  .desc
    text-align: left
    line-height: 36px
    font-size: 21px
    margin-right: 12px
  .number
    line-height: 40px
    font-size: 15px
    text-align: left
    font-weight: bold
    margin-right: 12px
  .icons
    margin-left: 2px
    font-size: 21px
    line-height: 37px
    color: #777777
    cursor: pointer

</style>

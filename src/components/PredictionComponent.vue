<template lang="pug">
  .prediction-component-container
    .prediction-title-container {{ name }}
    .prediction-display-container
      .prediction-viz-container(:id="refName")
      .prediction-info-wrapper
        .prediction-info-container
          .prediction-info-total-container(v-model="rateList") {{ `${rate}%` }}
          .prediction-info-element-list-wrapper
            .prediction-info-element-list-container
              .prediction-info-element-container(v-for="infoElement in infoElementList" v-bind="infoElement")
                .prediction-info-element-image-container
                  .prediction-info-element-image(:style="{ background: infoElement.category_color }")
                .prediction-info-element-desc {{ `${rateList[infoElement.category_category].toFixed(2)}%` }}
</template>

<script>

import * as d3 from 'd3';
import _ from 'lodash';
import category from '../../data/category.json';
import model from '../../data/model.json';
import linearSVC from '../../data/LinearSVC_modified.json';
import logisticRegression from '../../data/LogisticRegression_modified.json';
import perceptron from '../../data/Perceptron_modified.json';
import SGDClassifier from '../../data/SGDClassifier_modified.json';
import deepMultiPC from '../../data/DeepMultiPerceptronClassifier_modified.json';
import multiPC from '../../data/MultiPerceptronClassifier_modified.json';
import singlePC from '../../data/SinglePerceptronClassifier_modified.json';

export default {
  name: 'PredictionComponent',
  data() {
    return {
      infoElementList: [],
      margin: { top: 12, right: 12, bottom: 30, left: 30 },
      initializer: {
        xScale: null,
        yScale: null,
        line: null,
        svgLine: null,
        dataSet: [{ x: 0, y: 0 }]
      },
      modelData: {
        LinearSVC: linearSVC,
        LogisticRegression: logisticRegression,
        Perceptron: perceptron,
        SGDClassifier,
        DeepMultiPerceptronClassifier: deepMultiPC,
        MultiPerceptronClassifier: multiPC,
        SinglePerceptronClassifier: singlePC
      },
      xScale: null,
      yScale: null,
      width: null,
      height: null,
      name: model[this.refName],
      numOfCategories: 17,
      rate: 0,
      rateList: _.fill(Array(this.numOfCategories), 0),
      svg: null,
      // selectedCategoryStateList의 키는 category이다.
      selectedCategoryStateList: _.fill(Array(this.numOfCategories), false),
      wholeData: null, // 얘는 받아옴
      correctCountList: _.fill(Array(this.numOfCategories), 0),
      line: _.fill(Array(this.numOfCategories), null),
      svgLine: _.fill(Array(this.numOfCategories), null),
      dataSet: [[{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }]]
    };
  },
  props: {
    refName: {
      type: String,
      required: true
    }
  },
  created() {
    this.wholeData = this.modelData[model[this.refName]];
  },
  mounted() {
    this.initialize(_.fill(Array(this.numOfCategories), false));
  },
  computed: {
  },
  watch: {
  },
  methods: {
    toggleInfoElement(index, state) {
      if (state) {
        this.infoElementList.push({
          category_category: index,
          category_color: category[index].color,
          category_rate: this.rateList[index]
        });
      } else {
        const idx = _.findIndex(this.infoElementList, ele => ele.category_category === index);
        this.infoElementList.splice(idx, 1);
      }
    },
    resetInfoElement() {
      this.infoElementList = [];
    },
    initialize(state) {
      const that = this;
      that.xScale = null;
      that.yScale = null;
      that.width = null;
      that.height = null;
      that.svg = null;
      that.rate = 0;
      that.rateList = _.fill(Array(this.numOfCategories), 0);
      that.correctCountList = _.fill(Array(this.numOfCategories), 0);
      that.selectedCategoryStateList = state;
      that.line = _.fill(Array(that.numOfCategories), null);
      that.svgLine = _.fill(Array(that.numOfCategories), null);
      that.dataSet = [[{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }], [{ x: 0, y: 0 }]];
      that.initializer = {
        xScale: null,
        yScale: null,
        line: null,
        svgLine: null,
        dataSet: [{ x: 0, y: 0 }]
      };
      d3.select(`.prediction-viz-container[id=${that.refName}]`)
        .selectAll('*')
        .remove();
      that.width = 344 - that.margin.left - that.margin.right;
      that.height = 228 - that.margin.top - that.margin.bottom;

      that.xScale = d3.scaleLinear()
        .domain([0, that.initializer.dataSet.length - 1])
        .range([0, that.width]);
      that.yScale = d3.scaleLinear()
        .domain([0, 100])
        .range([that.height, 0]);

      that.initializer.xScale = d3.scaleLinear()
        .domain([0, that.initializer.dataSet.length - 1])
        .range([0, that.width]);
      that.initializer.yScale = d3.scaleLinear()
        .domain([0, 100])
        .range([that.height, 0]);

      that.svg = d3.select(`.prediction-viz-container[id=${that.refName}]`)
        .append('svg')
        .attr('width', that.width + that.margin.left + that.margin.right)
        .attr('height', that.height + that.margin.top + that.margin.bottom)
        .append('g')
        .attr('transform', `translate(${that.margin.left},${that.margin.top})`);

      that.svg.append('g')
        .classed('x', true)
        .classed('axis', true)
        .attr('transform', `translate(0,${that.height})`)
        .call(d3.axisBottom(that.initializer.xScale)
          .tickFormat((e) => {
            if (Math.floor(e) !== e) {
              return;
            }
            return e;
          }));
      that.svg.append('g')
        .classed('y', true)
        .classed('axis', true)
        .call(d3.axisLeft(that.initializer.yScale));

      for (let i = 0; i < that.selectedCategoryStateList.length; i += 1) {
        if (that.selectedCategoryStateList[i]) {
          that.appendEach(i);
        }
      }
    },
    appendEach(i) {
      const that = this;
      that.svg.append('g')
        .classed(`x_${category[i].className}`, true)
        .attr('transform', `translate(0,${that.height})`);
      that.svg.append('g')
        .classed(`y_${category[i].className}`, true)
        .call(d3.axisLeft(that.yScale));
      that.line[i] = d3.line()
        .x((d, ind) => that.xScale(ind))
        .y(d => that.yScale(d.y))
        .curve(d3.curveCatmullRom);
      that.svgLine[i] = that.svg.append('path')
        .datum(that.dataSet[i])
        .classed(`line_${category[i].className}`, true)
        .attr('d', that.line[i])
        .attr('fill', 'none')
        .attr('stroke', `${category[i].color}`)
        .attr('stroke-width', 3);
    },
    startRender(time, count) {
      const that = this;
      const xCor = that.initializer.dataSet.length;
      const xICor = that.initializer.dataSet.length;
      that.initializer.dataSet.push({ x: xICor, y: Math.floor(Math.random() * 100) });
      that.updateInitialize(time);
      for (let i = that.selectedCategoryStateList.length - 1; i >= 0; i -= 1) {
        if (i === 0) {
          let c = 0;
          for (let j = 1; j < this.numOfCategories; j += 1) {
            c += that.correctCountList[j];
          }
          that.correctCountList[i] = c;
          that.rateList[i] = (100 * that.correctCountList[i]) / ((this.numOfCategories - 1) * xCor);
        } else {
          that.correctCountList[i] += that.wholeData[i - 1][count % 200][1];
          that.rateList[i] = (100 * that.correctCountList[i]) / (xCor);
        }
        that.dataSet[i].push({ x: xCor, y: that.rateList[i] });
        if (that.selectedCategoryStateList[i]) { // true면 (선택되었으면)
          that.updateNode(i, time);
        }
        that.rate = that.rateList[0].toFixed(2);
      }
    },
    toggleChart(index, state) {
      const that = this;

      if (state) {
        that.selectedCategoryStateList[index] = true;
        that.appendEach(index);
      } else {
        that.selectedCategoryStateList[index] = false;
        that.line[index] = null;
        that.svgLine[index] = null;
        that.svg.selectAll(`.x_${category[index].className}`).remove();
        that.svg.selectAll(`.y_${category[index].className}`).remove();
        that.svg.selectAll(`.line_${category[index].className}`).remove();
      }
    },
    updateInitialize(time) {
      const that = this;
      that.initializer.xScale = d3.scaleLinear()
        .domain([0, that.initializer.dataSet.length - 1])
        .range([0, that.width]);
      const xAxisCall = d3.axisBottom();
      xAxisCall.scale(that.initializer.xScale);
      const t = d3.transition()
        .duration(time);
      const x = that.svg.selectAll('.x')
        .data(['dummy']);
      const newX = x.enter()
        .append('g')
        .classed('x', true)
        .classed('axis', true)
        .attr('transform', `translate(0,${that.height})`);
      x.merge(newX)
        .transition(t)
        .call(xAxisCall);
    },
    updateNode(index, time) {
      const that = this;
      that.xScale = d3.scaleLinear()
        .domain([0, that.initializer.dataSet.length - 1])
        .range([0, that.width]);
      const xAxisCall = d3.axisBottom();
      xAxisCall.scale(that.xScale);
      const t = d3.transition()
        .duration(time);
      const x = that.svg.selectAll(`.x_${category[index].className}`)
        .data(['dummy']);
      const newX = x.enter()
        .append('g')
        .classed(`x_${category[index].className}`, true)
        .attr('transform', `translate(0,${that.height})`);
      x.merge(newX).transition(t);
      that.svgLine[index].datum(that.dataSet[index])
        .transition(t)
        .attr('d', that.line[index]);
    }
  }
};
</script>

<style scoped lang="sass">
.prediction-component-container
  width: calc(100% / 3)
  height: 312px
  font: 12px Arial
  border-radius: 12px
  box-shadow: 3px 3px 3px 2px rgb(213, 222, 229)
  .prediction-title-container
    width: 100%
    height: 60px
    text-align: center
    line-height: 72px
    font-size: 24px
    font-family: 'Playfair Display', serif
  .prediction-display-container
    width: 100%
    height: calc(100% - 60px)
    padding: 12px
    display: flex
    .prediction-viz-container
      width: calc(100% - 120px)
      height: 100%
    .prediction-info-wrapper
      width: 120px
      height: 100%
      display: flex
      align-items: center
      .prediction-info-container
        width: 100%
        height: 174px
        border: 1px solid #9aa0a6
        border-radius: 4px
        padding: 6px 10px
        .prediction-info-total-container
          width: 100%
          height: 40px
          border-bottom: 1px solid #9aa0a6
          margin-bottom: 3px
          line-height: 40px
          font-size: 24px
        .prediction-info-element-list-wrapper
          width: 100%
          height: calc(100% - 40px)
          display: flex
          align-items: center
          overflow-y: hidden
          .prediction-info-element-list-container
            width: 100%
            height: auto
            .prediction-info-element-container
              width: 100%
              height: 24px
              display: flex
              .prediction-info-element-image-container
                width: 24px
                height: 100%
                display: flex
                align-items: center
                .prediction-info-element-image
                  width: 18px
                  height: 18px
                  border-radius: 6px
              .prediction-info-element-desc
                width: calc(100% - 24px)
                height: 100%
                text-align: left
                font-size: 12px
                line-height: 24px
                white-space: nowrap
                overflow: hidden
                text-overflow: ellipsis

.overlay
  fill: none
  pointer-events: all

.dot
  cursor: pointer

.focus circle
  fill: none
  stroke: steelblue


</style>

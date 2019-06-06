<template>
  <div ref="slider">
    <div ref="rail">
      <div ref="marks"></div>
      <div ref="process"></div>
      <div ref="handle"></div>
      <div ref="tooltip"></div>
    </div>
    <input ref="rangeInput" type="range">
  </div>
</template>




<script lang="ts">


import { Component, Prop, Watch, Vue } from "vue-property-decorator";

@Component({})
export default class SliderThree extends Vue {
  $refs!: {
    slider: HTMLDivElement,
    rail: HTMLDivElement,
    marks: HTMLDivElement,
    process: HTMLDivElement,
    handle: HTMLDivElement,
    tooltip: HTMLDivElement,
    rangeInput: HTMLInputElement
  }

  @Prop({ default: false})
  isDisabled!: boolean;

  @Prop({ default: true })
  isDraggable!: boolean;

  @Prop({ default: 1})
  interval!: number;

  @Prop({ default: null})
  data!: any[];

  @Prop({default: 0})
  value!: string[] | number[];

  @Prop({ default: 0 })
  min!: number;

  @Prop({ default: 100})
  max!: number;

  @Prop({ default: true })
  tooltip!: boolean;

  @Prop({ default: "" })
  prefix!: string;

  @Prop({ default: "" })
  suffix!: string;

  @Prop({ default: false})
  simpleTheme!: boolean;

  private isTowed: boolean = false;
  private sliderWidth: number = 0;

  private currentValue: any = 0;

  get numericValue() {
    return this.data ? this.data[this.currentValue] : this.currentValue;
  }
  set numericValue(newVal) {
    this.currentValue = newVal;
  }
























}






</script>



<style lang="less">

</style>
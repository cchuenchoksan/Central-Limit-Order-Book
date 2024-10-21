<template lang="">
  <div class="col-span-5">
    <div v-if="bidVol != null" class="bar-right text-right" :style="{ width: (bidVol * 100 / maxVol) + '%' }">{{bidVol}}</div>
  </div>
  <div class="col-span-1 text-center center-column right-border">{{bidPrice}}</div>
  <div class="col-span-1 text-center center-column">{{askPrice}}</div>
  <div class="col-span-5">
    <div v-if="askVol != null" class="bar-left" :style="{ width: (askVol * 100 / maxVol) + '%' }">{{askVol}}</div>
  </div>
</template>

<script>
export default {
  props: {
    bidVol: {
      default: 0,
    },
    bidPrice: {
      default: 0,
    },

    askVol: {
      default: 0,
    },
    askPrice: {
      default: 0,
    },

    maxVol: {
      default: 0,
    },
  },
};
</script>

<style>
.bar-left {
  background-color: red;
  color: blue;
  padding: 5px;
}

.bar-right {
  background-color: green;
  position: relative;
  float: right;
  color: blue;  
  padding: 5px;
}

.center-column {
  padding: 5px;
}

.right-border {
  border-right: 1px !important;
  border-right-color: black !important;
}
</style>

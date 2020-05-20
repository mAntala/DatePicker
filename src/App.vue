<template>
  <div id="app">
    <DatePicker v-model="myDate" :highlightToday="true" :range="true" :formatted="false" ref="picker"></DatePicker>
  </div>
</template>

<script>
import DatePicker from "@/components/DatePicker";

export default {
  name: "App",
  components: {
    DatePicker
  },
  data() {
    return {
      // myDate: new Date(1589278277000),
      myDate: [new Date(1589278277000), new Date(1589278277000)],
      // myDate: "15-5-2020"
    };
  },
  mounted() {
    // this.$refs.picker.setFormattedDate( "18-05-2020" );
    // console.log( this.$refs.picker.getFormattedDate() );
  }
};
</script>

<style lang="scss">
html, body {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>

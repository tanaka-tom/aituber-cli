<template>
  <div class="hello" v-bind:style="{ backgroundColor: '#' + color }">
    <p style="color: white;">{{ message }}</p>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class HelloWorld extends Vue {
  @Prop() public color!: string;
  @Prop() public message!: string;
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  width: 5em;
  height: 5em;
  text-align: center;
  border-radius: 50%;
  margin: auto;
}
</style>

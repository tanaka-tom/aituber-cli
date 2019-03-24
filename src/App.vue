<template>
  <div id="app">
    <HelloWorld
      :color="colorText"
      :message="messageText"
    />
    <YouTubeLiveComments
      :channel-id="channelId"
      :api-key="apiKey"
      @updated-comments="onUpdatedComments"
      @throw-error-message="displayErrorMessage"
    />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import HelloWorld from "./components/HelloWorld.vue";
import YouTubeLiveComments from "./components/YouTubeLiveComments.vue";
import * as Axios from "axios";

@Component({
  components: {
    HelloWorld,
    YouTubeLiveComments
  }
})
export default class App extends Vue {
  public colorText: string = "fff";
  public messageText: string = "";
  public channelId = "";
  public apiKey = "";

  public mounted() {
    this.channelId = process.env.VUE_APP_YOUTUBE_CHANNEL_ID;
    this.apiKey = process.env.VUE_APP_GOOGLE_API_KEY;
  }

  public displayErrorMessage(message: string) {
    this.messageText = message;
  }

  public onUpdatedComments(messages: Array<string>) {
    console.log(messages);
    if (messages.length >= 1) {
      this.messageText = messages[0];
      this.changeColorByMessage(this.messageText);
    }
  }

  public changeColorByMessage(message: string) {
    Axios.default
      .get("http://localhost:5000/analyze_sentiment", {
        headers: {
          "Access-Control-Allow-Origin": "*",
          "Content-Type": "application/json"
        },
        params: { text: message }
      })
      .then(response => {
        const label: string = response.data.result.label;
        let color: string;
        switch (label) {
          case "POSITIVE":
            color = "ff0";
            break;
          case "NEGATIVE":
            color = "f0f";
            break;
          default:
            color = "0ff";
            break;
        }
        this.colorText = color;
      });
  }
}
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
  background-color: #000;
}
</style>

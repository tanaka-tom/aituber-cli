<template></template>

<script lang="ts">
import * as Axios from "axios";
import { Component, Emit, Prop, Vue, Watch } from "vue-property-decorator";

@Component
export default class YouTubeLiveComments extends Vue {
  public lastMessageAt: number = Date.now();
  @Prop() public channelId!: string;
  @Prop() public apiKey!: string;

  @Watch("apiKey")
  onApiKeyChanged(_val: string, _oldVal: string) {
    this.fetchComment();
  }
  @Emit() public updatedComments(comments: Array<string>) {
    return comments;
  }
  @Emit() public throwErrorMessage(errorMessage: string) {
    return errorMessage;
  }
  public fetchComment = async () => {
    try {
      const searchResponse = await Axios.default.get(
        "https://www.googleapis.com/youtube/v3/search",
        {
          params: {
            key: this.apiKey,
            part: "id",
            channelId: this.channelId,
            type: "video",
            eventType: "live"
          }
        }
      );
      const searchResponseItems = searchResponse.data.items;
      if (searchResponseItems.length == 0) {
        const currentTime: string = new Date().toLocaleString();
        this.throwErrorMessage(
          `APIキー&チャンネルキーは正しく設定されていましたが、ライブ配信を取得できませんでした。YouTubeの仕様上開始後数分は取得できない可能性があります。10秒後に再取得を行います。:${currentTime}`
        );
        return setTimeout(() => {
          this.fetchComment();
        }, 10000);
      }
      const videoId: string = searchResponse.data.items[0].id.videoId;
      const videoResponse = await Axios.default.get(
        "https://www.googleapis.com/youtube/v3/videos",
        {
          params: {
            key: this.apiKey,
            part: "liveStreamingDetails,snippet",
            id: videoId
          }
        }
      );
      const liveChatId: string = await videoResponse.data.items[0]
        .liveStreamingDetails.activeLiveChatId;
      const messageResponse = await Axios.default.get(
        "https://www.googleapis.com/youtube/v3/liveChat/messages",
        {
          params: {
            key: this.apiKey,
            liveChatId: liveChatId,
            part: "snippet"
          }
        }
      );
      const messageItems = messageResponse.data.items;
      const targetMessages = messageItems.filter((element) => {
        return Date.parse(element.snippet.publishedAt) > this.lastMessageAt;
      });
      console.log(targetMessages);
      const messages = targetMessages.map((element) => {
        this.lastMessageAt = Date.parse(element.snippet.publishedAt);
        return element.snippet.displayMessage;
      });

      this.updatedComments(messages);
      return setTimeout(() => {
        this.fetchComment();
      }, 5000);
    } catch (error) {
      this.throwErrorMessage(error.response.data.error.errors[0].reason);
    }
  };
}
</script>

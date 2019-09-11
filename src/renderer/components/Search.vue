<template>
  <div id="app">
    <div class="container" style="max-width: 640px">
    <form id="search-form">
      <b-field>
        <b-input id="search-query" placeholder="검색" rounded expanded></b-input>
        <p class="control">
          <button class="button is-primary" type="submit">검색</button>
        </p>
      </b-field>
    </form>
    <ul id="search-result">
      <li v-for="item in items" v-bind:key="item">
        <div class="card" style="width: 640px; ">
        <div class="embed-responsive embed-responsive-16by9">
            <iframe id="ytplayer" class="embed-responsive-item" type="text/html" allowfullscreen
        :src="'http://www.youtube.com/embed/' + item.id.videoId"
        frameborder="0"/>
        </div>
          <div class="card-body">
            <h5 class="card-title">{{ item.snippet.title }}</h5>
            <p class="card-text">{{ item.snippet.description }}</p>
            <div class="btn-group">
                <button :onclick="download(item.id.videoId)" class="btn btn-primary">다운로드</button>
                <button :onclick="open('https://www.youtube.com/channel/' + item.snippet.channelId)" class="btn btn-secondary">{{ item.snippet.channelTitle }}</button>
            </div>
          </div>
        </div>
      </li>
    </ul>
    </div>
  </div>
</template>

<script>
var open = require('open');
export default {
  name: "search",
  components: {},
  data() {
    return {
        items: [],
    };
  },
  methods: {
      open(url) {
          console.log('외부 링크 열림');
      },    
      download(videoId)
      {

      }
  },
  mounted() {
    var ref = this;
    var query = "";
    var req = require("request");
    $("#search-form").submit(function(e) {
      e.preventDefault();
      ref.items = [];
      query = encodeURI($("#search-query").val());
      var params = {
        q: query,
        part: "snippet",
        key: "AIzaSyAIfzvhK5TpgtpXAuItLlKd-jjM3byoIOw",
        type: "video"
      };
      var url = "https://www.googleapis.com/youtube/v3/search?";
      for (var opIdx in params) {
        url += opIdx + "=" + params[opIdx] + "&";
      }
      url = url.substr(0, url.length - 1);
      req(url, function(err, res, body) {
        var ret = JSON.parse(body).items;
        for (var content in ret) {
          console.log(
            ret[content].snippet.title + " : " + ret[content].id.videoId
          );
          ref.items.push(ret[content]);
        }
      });
    });
  }
};
</script>

<style>
#app {
    margin: 0 auto;
}


</style>
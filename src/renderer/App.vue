<template>
  <div id="app">
    <div class="container" style="max-width: 800px">
      <form id="search-form">
        <b-field>
          <b-input id="search-query" placeholder="검색" rounded expanded></b-input>
          <p class="control">
            <button class="button is-primary" type="submit">검색</button>
          </p>
        </b-field>
      </form>
      <ul id="search-result">
        <li v-for="video in items" v-bind:key="video">
          <div class="card" style="width: 800px; ">
            <div class="embed-responsive embed-responsive-16by9"> 
              <iframe
                id="ytplayer"
                class="embed-responsive-item"
                type="text/html"
                allowfullscreen
                :src="'http://www.youtube.com/embed/' + video.id.videoId"
                frameborder="0"
              />
            </div>
            <div class="card-body">
              <h5 class="card-title">{{ video.snippet.title }}</h5>
              <p class="card-text">{{ video.snippet.description }}</p>
              <div class="btn-group">
                <button
                  v-on:click="downloadVideo(video.id.videoId,18)"
                  class="btn btn-primary"
                >영상 다운로드(저화질)</button>
                <button
                  v-on:click="downloadVideo(video.id.videoId,22)"
                  class="btn btn-primary"
                >영상 다운로드(고화질)</button>
                <button
                  v-on:click="downloadAudio(video.id.videoId)"
                    class="btn btn-primary"
                >음성 다운로드</button>
                <button
                  v-on:click="open('https://www.youtube.com/channel/' + video.snippet.channelId)"
                  class="btn btn-secondary"
                >{{ video.snippet.channelTitle }}</button>
              </div>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "youtube-dl",
  components: {},
  data() {
    return {
      items: [],
      quality: "",
    };
  },
  methods: {
    open(url) {
      console.log("외부 링크 열림");
      open(url);
    },
    noti(msg) {
      this.$buefy.toast.open({
          message: msg,
          type: 'is-info',
          position: 'is-bottom-right',
          duration: 2500
        });
    },
    downloadVideo(videoId, format) {
      var self = this;
      var fs = require('fs');
      var dl = require('youtube-dl');
      var path = require('path');
      var video = dl("http://www.youtube.com/watch?v=" + videoId , ['--format=' + format], {cwd: __dirname});
      var filename = "";
      if(!fs.existsSync(process.cwd() + '/Videos'))
      {
        fs.mkdirSync(process.cwd() + '/Videos');
      }
      video.on('info', function(info) {
        self.noti('영상 다운로드 시작 - ' + info._filename);
        filename = info._filename;
      });
      video.on('end', function() {
        self.noti('영상 다운로드 완료 - ' + filename);
      });
      video.pipe(fs.createWriteStream(path.join(process.cwd() + '/Videos' ,videoId + ".mp4")));
    },
    downloadAudio(videoId) {
      var self = this;
      var fs = require('fs');
      var path = require('path');
      var dl = require('youtube-dl');
      var audio = dl("http://www.youtube.com/watch?v=" + videoId, ['--extract-audio', '--audio-format=mp3', '--audio-quality=0']);
      var filename;
      if(!fs.existsSync(process.cwd() + '/Audios'))
      {
        fs.mkdirSync(process.cwd() + '/Audios');
      }
      audio.on('info', function(info){
        self.noti('음성 다운로드 시작 - ' + info._filename);
        filename = info._filename;
      });
      audio.on('end', function() {
        self.noti('음성 다운로드 완료 - ' + filename);
      });
      audio.pipe(fs.createWriteStream(path.join(process.cwd() + '/Audios', videoId + '.mp3')));
    }
  },
  mounted() {
    var ref = this;
    var query = "";
    var req = require("request");
    var count = "";
    var nextToken = "";
    
    $("#search-form").submit(function(e) {
      e.preventDefault();
      console.log('검색 시작');
      ref.items = [];
      nextToken = "";
      query = encodeURI($("#search-query").val());
      var params = {
        q: query,
        part: "snippet",
        key: "AIzaSyAIfzvhK5TpgtpXAuItLlKd-jjM3byoIOw",
        type: "video",
      };
      var url = "https://www.googleapis.com/youtube/v3/search?";
      for (var opIdx in params) {
        url += opIdx + "=" + params[opIdx] + "&";
      }
      url = url.substr(0, url.length - 1);
      req(url, function(err, res, body) {
        nextToken = JSON.parse(body).nextPageToken;
        var ret = JSON.parse(body).items;
        for (var content in ret) {
          console.log(
            ret[content].snippet.title + " : " + ret[content].id.videoId
          );
          ref.items.push(ret[content]);
        }
      });
    });

    $(window).scroll(function() {
      if($(window).scrollTop() == ($(document).height() - $(window).height()))
      {
        var params = {
        q: query,
        part: "snippet",
        key: "AIzaSyAIfzvhK5TpgtpXAuItLlKd-jjM3byoIOw",
        type: "video",
        pageToken: nextToken
      };
      var url = "https://www.googleapis.com/youtube/v3/search?";
      for (var opIdx in params) {
        url += opIdx + "=" + params[opIdx] + "&";
      }
      url = url.substr(0, url.length - 1);
      req(url, function(err, res, body) {
        nextToken = JSON.parse(body).nextPageToken;
        var ret = JSON.parse(body).items;
        for (var content in ret) {
          console.log(
            ret[content].snippet.title + " : " + ret[content].id.videoId
          );
          ref.items.push(ret[content]);
        }
      });
      }
    });
  }
};
</script>

<style>
@import "https://cdn.materialdesignicons.com/2.5.94/css/materialdesignicons.min.css";
aside {
  margin: 20px;
  width: 225px;
}

#search-form {
  margin-top: 100px;
}
</style>

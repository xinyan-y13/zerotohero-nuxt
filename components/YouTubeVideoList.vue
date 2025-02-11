<template>
  <container-query :query="query" v-model="params">
    <div class="youtube-video-list">
      <client-only>
        <div
          v-if="$adminMode"
          class="mb-4 youtube-video-list-admin-bar rounded p-3 w-100"
        >
          <div>
            <b-button
              class="mt-1 mb-1"
              v-if="!checkSavedData"
              size="sm"
              @click="checkSavedData = true"
            >
              <i class="fas fa-question mr-2"></i>
              Check Saved
            </b-button>
            <b-button
              class="mt-1 mb-1"
              v-if="checkSavedData"
              size="sm"
              @click="checkSavedData = false"
            >
              <i class="fas fa-question mr-2"></i>
              Uncheck Saved
            </b-button>
            <b-button
              class="mt-1 mb-1"
              variant="secondary"
              v-if="checkSavedData"
              size="sm"
              @click="addAll()"
            >
              <i class="fas fa-plus mr-2"></i>
              Add All
            </b-button>
            <b-button size="sm" @click="surveyChannels">
              Survey Channels
            </b-button>
            <b-button class="mt-1 mb-1" @click="removeAll()" size="sm">
              <i class="fas fa-trash mr-2"></i>
              Remove All
            </b-button>
            <br />
          </div>
          <div>
            <AssignShow
              size="sm"
              @assignShow="assignShowToAll"
              :defaultSelection="keyword"
              type="tv-shows"
            />
            <AssignShow
              size="sm"
              @assignShow="assignShowToAll"
              :defaultSelection="keyword"
              type="talks"
            />
            <drop
              @drop="handleDrop"
              :class="{
                'subs-drop text-secondary rounded d-inline-block text-center mt-1': true,
                over: over,
                drop: true,
              }"
              :key="`drop-${_uid}`"
              @dragover="over = true"
              @dragleave="over = false"
              style="font-size: 0.9em"
            >
              Drop SRT files here to bulk add subs ...
            </drop>
          </div>
          <div class="mt-1" style="font-size: 0.9em">
            <b-form-checkbox
              class="mr-1 d-inline-block"
              v-model="hideVideosWithoutSubs"
            >
              Hide Videos without Subs
            </b-form-checkbox>
            <b-form-checkbox
              class="mr-1 d-inline-block"
              v-model="hideVideosInShows"
            >
              Hide Videos in Shows
            </b-form-checkbox>
            <b-form-checkbox
              class="mr-1 d-inline-block"
              v-model="showSubsEditing"
            >
              Show Subs Editing
            </b-form-checkbox>
            <a
              class="link-unstyled"
              @click="removeAllUnavailable()"
              v-if="isLocalHost()"
            >
              Remove unavailable videos
            </a>
          </div>
          <div v-if="channels">
            <h6 class="mt-2">Channels with the most videos below:</h6>
            <div
              v-for="(channel, index) in channels"
              :key="`video-list-unique-by-channel-${index}`"
            >
              {{ channel.videos.length }} Videos:
              <router-link
                :to="{
                  name: 'youtube-view',
                  params: { youtube_id: channel.videos[0].youtube_id },
                }"
              >
                {{ channel.videos[0].title.slice(0,30) }}
              </router-link>
            </div>
          </div>
        </div>
      </client-only>
      <div class="youtube-videos row">
        <div
          v-for="(video, videoIndex) in filteredVideos"
          :class="{
            'col-sm-12': view === 'list' || singleColumn,
            'col-12': params.xs && view === 'grid' && !singleColumn,
            'col-6': params.sm && view === 'grid' && !singleColumn,
            'col-4': params.md && view === 'grid' && !singleColumn,
            'col-3':
              (params.lg || params.xl) && view === 'grid' && !singleColumn,
          }"
          :style="`padding-bottom: ${view === 'list' ? '1rem' : '2rem'}`"
          :key="`youtube-video-wrapper-${video.youtube_id}-${videoIndex}`"
        >
          <YouTubeVideoCard
            ref="youTubeVideoCard"
            @newShow="newShow"
            @unavailable="onVideoUnavailable"
            :video="video"
            :checkSubs="checkSubsData"
            :showSubsEditing="showSubsEditing"
            :checkSaved="checkSavedData"
            :view="view"
            :showBadges="showBadges"
            :showDate="showDate"
            :skin="skin"
            :showProgress="showProgress"
            :showPlayButton="showPlayButton"
          />
        </div>
      </div>
    </div>
  </container-query>
</template>

<script>
import Vue from "vue";
import YouTube from "@/lib/youtube";
import Helper from "@/lib/helper";
import Config from "@/lib/config";
import { Drag, Drop } from "vue-drag-drop";
import { ContainerQuery } from "vue-container-query";

export default {
  components: {
    ContainerQuery,
    Drag,
    Drop,
  },
  props: {
    videos: {
      type: Array,
    },
    keyword: {
      type: String,
    },
    checkSaved: {
      type: Boolean,
      default: false,
    },
    checkSubs: {
      type: Boolean,
      default: false,
    },
    view: {
      type: String,
      default: "grid",
    },
    singleColumn: {
      type: Boolean,
      default: false,
    },
    showBadges: {
      default: false,
    },
    showDate: {
      default: false,
    },
    skin: {
      default: "card", // or 'dark'
    },
    showProgress: {
      default: false,
    },
    showPlayButton: {
      default: false,
    },
  },

  data() {
    return {
      Helper,
      videosInfoKey: 0,
      channels: undefined,
      showSubsEditing: false,
      checkSavedData: this.checkSaved,
      checkSubsData: this.checkSubs,
      over: false,
      hideVideosWithoutSubs: false,
      showChannels: false,
      hideVideosInShows: false,
      unavailableYouTubeIds: [],
      params: {},
      query: {
        xs: {
          minWidth: 0,
          maxWidth: 423,
        },
        sm: {
          minWidth: 423,
          maxWidth: 720,
        },
        md: {
          minWidth: 720,
          maxWidth: 960,
        },
        lg: {
          minWidth: 960,
          maxWidth: 1140,
        },
        xl: {
          minWidth: 1140,
        },
      },
    };
  },
  computed: {
    $l1() {
      if (typeof this.$store.state.settings.l1 !== "undefined")
        return this.$store.state.settings.l1;
    },
    $l2() {
      if (typeof this.$store.state.settings.l2 !== "undefined")
        return this.$store.state.settings.l2;
    },
    $adminMode() {
      if (typeof this.$store.state.settings.adminMode !== "undefined")
        return this.$store.state.settings.adminMode;
    },
    filteredVideos() {
      let filteredVideos = this.videos.filter((video) => {
        if (this.hideVideosWithoutSubs && !video.hasSubs) return false;
        if (
          this.hideVideosInShows &&
          video.id &&
          ((video.tv_show && video.tv_show.id) || (video.talk && video.talk.id))
        )
          return false;

        return true;
      });
      if (this.isLocalHost()) {
        filteredVideos = filteredVideos.filter((video) => {
          if (!this.$adminMode && video.unavailable) return false;
          return true;
        });
      }
      return filteredVideos;
    },
  },
  watch: {
    async checkSavedData() {
      if (this.checkSavedData) {
        await this.checkSavedFunc(this.filteredVideos);
      } else {
        for (let video of this.filteredVideos) {
          delete video.tv_show;
          delete video.talk;
          Vue.delete(video, "id");
        }
      }
    },
  },
  methods: {
    isLocalHost() {
      return (
        typeof window !== "undefined" &&
        window.location.href.startsWith("http://localhost")
      );
    },
    onVideoUnavailable(youtube_id) {
      this.unavailableYouTubeIds.push(youtube_id);
    },
    surveyChannels() {
      let groups = Helper.groupArrayBy(this.videos, "channel_id");
      let channels = [];
      for (let channel_id in groups) {
        channels.push({
          channel_id,
          videos: groups[channel_id],
        });
      }
      channels = channels.sort((a, b) => b.videos.length - a.videos.length);
      this.channels = channels;
    },
    async checkSavedFunc(videos) {
      videos = videos.filter((v) => !v.id); // Only check those that are not saved
      let youtube_ids = videos
        .map((v) => v.youtube_id)
        .filter((id) => !id.includes("0x"));
      let chunks = Helper.arrayChunk(youtube_ids, 100);
      for (let youtube_ids of chunks) {
        let response = await axios.get(
          `${
            Config.wiki
          }items/youtube_videos?filter[youtube_id][in]=${youtube_ids}&fields=id,title,channel_id,youtube_id,tv_show.*,talk.*${
            this.showSubsEditing ? ",subs_l2" : ""
          }&filter[l2][eq]=${this.$l2.id}&timestamp=${
            this.$adminMode ? Date.now() : 0
          }`
        );
        if (response.data && response.data.data && response.data.data[0]) {
          let savedVideos = response.data.data;
          for (let video of videos) {
            let savedVideo = savedVideos.find(
              (v) => v.youtube_id === video.youtube_id
            );
            if (savedVideo) {
              video.tv_show = savedVideo.tv_show;
              video.talk = savedVideo.talk;
              if (savedVideo.subs_l2) {
                let subs_l2 = YouTube.parseSavedSubs(savedVideo.subs_l2);
                if (subs_l2[0]) {
                  video.subs_l2 = subs_l2;
                  this.firstLineTime = video.subs_l2[0].starttime;
                }
              }
              Vue.set(video, "id", savedVideo.id);
            }
          }
        }
      }
    },
    newShow(show) {
      this.$emit("newShow", show);
    },
    async batch(action) {
      let indices = Object.keys(this.$refs.youTubeVideoCard);
      let chunks = Helper.arrayChunk(indices, 3);
      for (let chunk of chunks) {
        let promises = [];
        for (let videoIndex of chunk) {
          promises.push(action(videoIndex));
        }
        await Promise.all(promises);
      }
    },
    async addAll() {
      this.batch((videoIndex) => {
        if (this.$refs.youTubeVideoCard[videoIndex])
          this.$refs.youTubeVideoCard[videoIndex].getSubsAndSave();
      });
    },
    async assignShowToAll(show, type) {
      // type: 'tv_show' or 'talk'
      this.batch((videoIndex) => {
        if (this.$refs.youTubeVideoCard[videoIndex])
          this.$refs.youTubeVideoCard[videoIndex].saveShow(show, type);
      });
    },
    async removeAll() {
      for (let videoIndex in this.$refs.youTubeVideoCard) {
        await this.$refs.youTubeVideoCard[videoIndex].remove();
      }
    },
    async removeAllUnavailable() {
      this.unavailableYouTubeIds = Helper.unique(this.unavailableYouTubeIds);
      for (let videoIndex in this.$refs.youTubeVideoCard) {
        let videoCard = this.$refs.youTubeVideoCard[videoIndex];
        if (this.unavailableYouTubeIds.includes(videoCard.video.youtube_id)) {
          await videoCard.remove();
        }
      }
    },
    handleDrop(data, event) {
      event.preventDefault();
      let files = event.dataTransfer.files;
      this.importSrtToAll(files);
    },
    importSrtToAll(files) {
      for (let file of files) {
        for (let videoIndex in this.$refs.youTubeVideoCard) {
          let card = this.$refs.youTubeVideoCard[videoIndex];
          let numsInFileName = file.name.match(/\d+/g);
          let numsInVideoTitle = card.video.title.match(/\d+/g);
          let found = false;
          if (numsInFileName && numsInVideoTitle) {
            for (let n of numsInFileName) {
              for (let m of numsInVideoTitle) {
                if (Number(n) === Number(m)) {
                  found = true;
                }
              }
            }
            if (found !== false) {
              card.importSrt(file);
            }
          }
        }
      }
    },
  },
};
</script>

<style lang="scss">
.youtube-video-list-admin-bar {
  background: rgb(205, 207, 212);
}

.main-dark {
  .youtube-video-list-admin-bar {
    background-color: #88888822;
  }
}

.subs-drop {
  border: 1px #666 dashed;
  padding: 0.2rem 0.6rem;
}
</style>

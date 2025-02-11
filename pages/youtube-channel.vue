<router>
  {
    path: '/:l1/:l2/youtube/channel/:channel_id?/:title?',
    props: true,
    meta: {
      title: 'Study YouTube Subtitles | Zero to Hero',
      metaTags: [
        {
          name: 'description',
          content: 'Watch YouTube videos and study the subtitles.'
        }
      ]
    }
  }
</router>
<template>
  <div class="main">
    <div class="youtube-browse container pt-5 pb-5">
      <div class="row">
        <div class="col-sm-12">
          <h3 class="mb-4 text-center">
            <Annotate :phonetics="false" :buttons="true" v-if="title">
              <span>{{ title }}</span>
            </Annotate>
            <span v-else>YouTube Channel {{ channel_id }}</span>
          </h3>
          <div class="text-center">
            <b-button
              class="btn-small btn-primary d-inline-block"
              @click="forceRefresh"
            >
              <i class="fa fa-sync-alt mr-1"></i>
              Force Refresh
            </b-button>
          </div>
          <template v-if="!loading">
            <h4 class="text-center mt-5">{{ $t("Playlists") }}</h4>
            <hr />
            <YouTubePlaylists :playlists="playlists" />
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import YouTube from "@/lib/youtube";

export default {
  props: {
    channel_id: {
      type: String,
    },
    title: {
      type: String,
    },
  },
  data() {
    return {
      playlists: [],
      videos: [],
      loading: true,
      saved: false,
    };
  },
  mounted() {
    this.loadChannel();
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
  },
  methods: {
    forceRefresh() {
      this.loadChannel({ forceRefresh: true });
    },
    async loadChannel(options) {
      options = options || {};
      options = Object.assign(
        {
          forceRefresh: false,
        },
        options
      );
      this.loading = true;
      this.videos = [];
      let playlists = await YouTube.channelPlayListsByAPI(
        this.channel_id,
        options.forceRefresh ? 0 : -1
      );
      if (playlists) {
        playlists = playlists.sort((a, b) => b.count - a.count);
        let allVideosPlaylist = {
          id: this.channel_id.replace("UC", "UU"),
          title: "All uploaded videos of " + (this.title ? this.title : 'this channel'),
        };
        if (playlists[0]) allVideosPlaylist.thumbnail = playlists[0].thumbnail;
        playlists = [allVideosPlaylist, ...playlists];
        this.playlists = playlists;
      }
      this.loading = false;
    },
  },
};
</script>

<style>
.avatar {
  border-radius: 100%;
  height: 4rem;
}
</style>

<template>
  <div ref="player"></div>
</template>

<script>
export default {
  name: "twitch-vue",
  data() {
    return {
      player: null,
    };
  },
  props: {
    channel: String,
    width: {
      type: Number,
      default: 850,
    },
    height: Number,
    volume: {
      type: Number,
      default: 1,
    },
    autoplay: {
      type: Boolean,
      default: true,
    },
  },
  computed: {
    computedHeight() {
      return this.width / (16 / 9);
    },
    computedWidth() {
      return this.height * (16 / 9);
    },
  },
  mounted() {
    function load(src) {
      return new Promise(function(resolve, reject) {
        let shouldAppend = false;
        let el = document.querySelector(`script[src="${src}"]`);
        if (!el) {
          el = document.createElement("script");
          el.type = "text/javascript";
          el.async = true;
          el.src = src;
          shouldAppend = true;
        } else if (el.hasAttribute("data-loaded")) {
          resolve(el);
          return;
        }

        el.addEventListener("error", reject);
        el.addEventListener("abort", reject);
        el.addEventListener("load", function loadScriptHandler() {
          el.setAttribute("data-loaded", true);
          resolve(el);
        });

        if (shouldAppend) document.head.appendChild(el);
      });
    }

    load("https://player.twitch.tv/js/embed/v1.js")
      .then(() => {
        const options = {};
        if (this.width && this.height) {
          // Clean this up you fool
          options.width = this.width;
          options.height = this.height;
        } else {
          // 😬
          if (this.width && !this.height) {
            options.width = this.width;
            options.height = this.computedHeight;
          } else if (!this.width && this.height) {
            options.width = this.computedWidth;
            options.height = this.height;
          } else {
            options.width = 800;
            options.height = 450;
          }
        }
        options.muted = false;
        options.autoplay = this.autoplay;
        this.channel
          ? (options.channel = this.channel)
          : console.log("error", "no source specified");
        this.player = new window.Twitch.Player(this.$refs.player, options);

        this.player.addEventListener("ready", () => {
          this.player.setVolume(1);
          this.player.setMuted(false);
        });
      })
      .catch((e) => console.log("error", e));
  },
};
</script>

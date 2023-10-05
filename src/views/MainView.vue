<template>
  <div class="main">
    <div class="item">
      <input
        type="file"
        @change="changeFile($event.target as HTMLInputElement)"
        accept=".mp4"
      >
    </div>
    <div class="item dimentions">
      <video
        id="main_video"
        ref="main_video"
        @loadeddata="loaded"
        controls
      ></video>
      <audio id="main_audio" controls></audio>
      <span class="video_width">{{ videoWidth }}</span>
      <span class="video_height">{{ videoHeight }}</span>
    </div>
    <div class="item">

    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

interface MainData {
  mainFile: File | null,
  videoHeight: number,
  videoWidth: number,
}

export default defineComponent({
  name: 'MainView',
  data () {
    return {
      mainFile: null,
      videoHeight: 0,
      videoWidth: 0,
    } as MainData
  },
  methods: {
    changeFile (input: HTMLInputElement): void {
      if (input) {
        const files = input.files as FileList

        this.mainFile = !files.length ? null : files[0]
      }
    },
    clearFile (component: string): void {
      const comp = this.$refs[component] as HTMLVideoElement
        
      if (!comp.paused) {
        comp.pause()
        return this.clearFile(component)
      }

      comp.removeAttribute('src')
      comp.load()
    },
    loadFile (component: string) {
      if (this.mainFile)
        this.clearFile(component)

      const comp = this.$refs[component] as HTMLVideoElement

      const reader = new FileReader()

      reader.addEventListener('load', (event: Event) => {
        const self = event.target as FileReader

        const buffer = self.result as ArrayBuffer,
              blob   = new Blob([new Uint8Array(buffer)], { type: 'video/mp4' }),
              url    = window.URL.createObjectURL(blob)

        comp.setAttribute('src', url)
      })

      reader.readAsArrayBuffer(this.mainFile as File)
    },
    loaded () {
      const video = this.$refs['main_video'] as HTMLVideoElement

      this.videoHeight = video.videoHeight
      this.videoWidth = video.videoWidth
    },
    extractAudio (buffer: ArrayBuffer) {
      console.log('extract audio from video');
    }
  },
  watch: {
    mainFile (val: File | null) {
      if (val) {
        this.loadFile('main_video')
      } else {
        this.clearFile('main_video')
      }
    }
  }
});
</script>

<style lang="scss">
.main {
  height: calc(100% - 3rem);
  width: calc(100% - 3rem);
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding: 1.5rem;

  .item {
    width: 100%;

    &:not(:last-child) {
      margin-bottom: 1rem;
    }

    &.dimentions {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      position: relative;
      padding-bottom: 1.5rem;

      .video_width {
        position: absolute;
        bottom: 0;
        left: calc(50% - 4.45px);
      }

      .video_height {
        position: absolute;
        top: calc(50% - 9px);
        right: 0;
      }
    }
  }
}

#main_video {
  width: 60%;
  height: auto;
}

#main_audio {
  width: 60%;
  margin-top: .5rem;
}

@media only screen and (max-width: 992px) {
  #main_video {
    width: 80%;
  }

  #main_audio {
    width: 80%;
  }
}
  
@media only screen and (max-width: 768px) {
  #main_video {
    width: calc(100% - 3rem);
  }

  #main_audio {
    width: calc(100% - 3rem);
  }
}
</style>
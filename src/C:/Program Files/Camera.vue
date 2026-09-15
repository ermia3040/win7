```vue
<template>
  <div :class="$style.camera">
    <div class="video">
      <video
        ref="video"
        muted
        autoplay
        playsinline
      />

      <span
        class="button"
        @click="takePhoto"
      >
        Take Photo
      </span>

      <span
        class="toggle"
        @click="openDirectory"
      >
        Open Taked Photos Directory
      </span>
    </div>
  </div>
</template>

<script>
import { rgba } from '@/styles/utils';
import { openFile } from '@/services/wm';
import {
  resolveFileByPath,
  createNewFile,
  fileObject,
} from '@/services/fs';
import { playBackgroundSound } from '@/services/snd';

const CAMERA_ICON = 'C:/Windows/system/icons/camera.png';
const CAMERA_SOUND = 'C:/Windows/system/sounds/notif.mp3';
const PICTURES_DIRECTORY = 'C:/User/Pictures';

export default {
  canHandle: (file) => !file,

  metaData: () => ({
    icon: resolveFileByPath(CAMERA_ICON),
    width: 600,
    height: 500,
    title: 'Camera',
  }),

  data() {
    return {
      stream: null,
    };
  },

  mounted() {
    this.startCamera();
  },

  beforeUnmount() {
    this.stopCamera();
  },

  methods: {
    async startCamera() {
      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        console.error('Camera API is not supported by this browser.');
        return;
      }

      try {
        this.stream = await navigator.mediaDevices.getUserMedia({
          audio: false,
          video: {
            facingMode: 'user',
          },
        });

        const video = this.$refs.video;

        if (video) {
          video.srcObject = this.stream;
        }
      } catch (error) {
        console.error('Unable to access camera:', error);
      }
    },

    stopCamera() {
      if (!this.stream) {
        return;
      }

      this.stream.getTracks().forEach((track) => {
        track.stop();
      });

      this.stream = null;

      if (this.$refs.video) {
        this.$refs.video.srcObject = null;
      }
    },

    takePhoto() {
      const video = this.$refs.video;

      if (!video || !video.videoWidth || !video.videoHeight) {
        console.warn('Camera is not ready yet.');
        return;
      }

      const canvas = document.createElement('canvas');

      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      const context = canvas.getContext('2d');

      if (!context) {
        console.error('Unable to create canvas context.');
        return;
      }

      context.drawImage(
        video,
        0,
        0,
        canvas.width,
        canvas.height
      );

      const fileName = `Photo ${Date.now()}.jpg`;
      const filePath = `${PICTURES_DIRECTORY}/${fileName}`;

      const data = canvas.toDataURL('image/jpeg', 0.92);

      const photo = fileObject(
        filePath,
        'image',
        data
      );

      createNewFile(photo);

      playBackgroundSound(
        resolveFileByPath(CAMERA_SOUND)
      );
    },

    openDirectory() {
      openFile(
        fileObject(
          PICTURES_DIRECTORY,
          'directory'
        )
      );
    },
  },

  style({ className }) {
    return [
      className('camera', {
        position: 'relative',

        '& > .video': {
          position: 'relative',
          width: '100%',
          height: '100%',
          overflow: 'hidden',

          '& > video, & > img': {
            width: '100%',
            height: '100%',
            border: 'none',
            resize: 'none',
            objectFit: 'cover',
            display: 'block',
          },

          '& > .toggle': {
            position: 'absolute',
            bottom: '0',
            left: '50%',
            transform: 'translateX(-50%)',
            padding: '2px 15px',
            borderTopRightRadius: '15px',
            borderTopLeftRadius: '15px',
            color: '#111',
            background: `linear-gradient(
              180deg,
              #eee 0%,
              #aaa 100%
            )`,
            border: 'solid 1px #ddd',
            cursor: 'pointer',
            userSelect: 'none',

            '&:hover': {
              filter: 'brightness(1.1)',
            },

            '&:active': {
              filter: 'brightness(0.9)',
            },
          },

          '& > .button': {
            position: 'absolute',
            bottom: '35px',
            left: '50%',
            transform: 'translateX(-50%)',
            border: '0',
            borderRadius: '20px',
            padding: '15px',
            color: '#fff',
            fontWeight: 'bold',
            cursor: 'pointer',
            userSelect: 'none',

            background: `linear-gradient(
              180deg,
              ${rgba([200, 0, 0], 0.8)} 0%,
              ${rgba([214, 0, 0], 0.8)} 47%,
              ${rgba([132, 0, 0], 0.8)} 53%,
              ${rgba([170, 0, 0], 0.8)} 100%
            )`,

            '&:not(.disabled):hover, &:not(.disabled):focus': {
              filter: 'brightness(1.2)',
            },

            '&:not(.disabled):active': {
              filter: 'brightness(0.8)',
            },

            '&.disabled': {
              filter: 'grayscale(1)',
              cursor: 'default',
            },
          },
        },
      }),
    ];
  },
};
</script>
```

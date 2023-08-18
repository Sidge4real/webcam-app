<template>
  <div class="app-container">
    <video ref="videoElement" autoplay></video>
    <button @click="takePhoto" :disabled="photoList.length >= 2">Take Photo</button>
    <div class="photo-list">
      <img
        v-for="(photoDataUrl, index) in photoList"
        :key="index"
        :src="photoDataUrl"
        alt="Taken Photo"
        @click="showFullscreen(index)"
      />
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';

export default {
  name: 'App',
  setup() {
    const videoElement = ref(null);
    const photoList = ref([]);

    const startCamera = async () => {
      try {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const videoDevices = devices.filter(device => device.kind === 'videoinput');
        const selectedDevice = videoDevices[0];

        const constraints = { video: { deviceId: selectedDevice.deviceId } };
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        videoElement.value.srcObject = stream;
      } catch (error) {
        console.error('Error accessing camera:', error);
      }
    };

    const takePhoto = () => {
      if (photoList.value.length < 2) {
        const canvas = document.createElement('canvas');
        const context = canvas.getContext('2d');
        canvas.width = videoElement.value.videoWidth;
        canvas.height = videoElement.value.videoHeight;
        context.drawImage(videoElement.value, 0, 0, canvas.width, canvas.height);
        const dataUrl = canvas.toDataURL('image/jpeg');
        photoList.value.push(dataUrl);
      }
    };

    const showFullscreen = index => {
      const img = new Image();
      img.src = photoList.value[index];
      const viewer = window.open('', '_blank');
      viewer.document.write(img.outerHTML);
    };

    onMounted(() => {
      startCamera();
    });

    return {
      videoElement,
      photoList,
      takePhoto,
      showFullscreen,
    };
  },
};
</script>

<style scoped>
.app-container {
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
video {
  width: 50%;
  height: 50%;
}
.photo-list {
  display: flex;
  flex-wrap: wrap;
}
.photo-list img {
  max-width: 100px;
  height: auto;
  margin: 5px;
  cursor: pointer;
}
</style>

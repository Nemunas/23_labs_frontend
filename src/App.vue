<template>
  <main>    
    <video ref="videoElement" autoplay playsinline style="position: absolute; width: 100%; height: 100%; object-fit: cover;"></video>
    <div class="content" style="position: fixed; bottom: 64px; width:100%; z-index: 1;">
      <div class="card" style="width:260px; margin:auto;">
        <div style="display:flex; flex-direction:column; padding-bottom:12px;">
          Please select a target language:
          <select v-model="selectedLanguage" style="margin-top:4px">
            <option value="">Please select a language</option>
            <option v-for="language in languages" :key="language" :value="language">
              {{ language }}
            </option>
          </select>      
        </div>
        <button @click="captureImage(); getResults()" style="width:100%">Get Results</button>            
      </div>
    </div>
  <canvas ref="canvas" style="display: none;"></canvas> 
  </main>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';

export default {
  setup() {
    const backendURL = 'http://localhost:3000/api';
    const selectedImage = ref(null);
    const getResults = async () => {
      if (!selectedImage.value || !this.selectedLanguage) {
        alert('Please select a language and upload an image');
        return;
      }
      let formData = new FormData();
      formData.append('language', this.selectedLanguage);
      formData.append('image', selectedImage.value);
      try {
        const response = await axios.post(backendURL, formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        });
        console.log(response.data);
      } catch (error) {
        console.error(error);
      }
    };
    const videoElement = ref(null);
    let stream = null;

    const initCamera = async () => {
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: true });
        videoElement.value.srcObject = stream;
      } catch (err) {
        console.error("Error accessing the camera:", err);
      }
    };

    onMounted(initCamera);

    onBeforeUnmount(() => {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
      }
    });

    const captureImage = () => {
      const canvas = this.$refs.canvas;
      const video = this.$refs.videoElement;
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      canvas.getContext('2d').drawImage(video, 0, 0);
      canvas.toBlob(blob => {
        selectedImage.value = new File([blob], "image.jpg", { type: "image/jpeg" });
      }, 'image/jpeg');
    };

    return { getResults, selectedImage, videoElement, captureImage };
  },
  data() {
    return {
      languages_list: [        
        "English",
        "Japanese",
        "Chinese",
        "German",
        "Hindi",
        "French",
        "Korean",
        "Portuguese",
        "Italian",
        "Spanish",
        "Indonesian",
        "Dutch",
        "Turkish",
        "Filipino",
        "Polish",
        "Swedish",
        "Bulgarian",
        "Romanian",
        "Arabic",
        "Czech",
        "Greek",
        "Finnish",
        "Croatian",
        "Malay",
        "Slovak",
        "Danish",
        "Tamil",
        "Ukrainian"
      ],
      selectedLanguage: "English"
    }
  } 
}
</script>

<style>

.card {
  /* position: absolute; */
  background-color: #FFFFFFAA;  
  border-radius: 10px;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
  padding: 8px;
  color:black;  
  border: 2px solid black;
}


</style>

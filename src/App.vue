<template>
  <main>    
    <video ref="videoElement" autoplay playsinline style="position: absolute; width: 100%; height: 100%; object-fit: cover;"></video>
    <div class="content" style="position: fixed; bottom: 64px; width:100%; z-index: 1;">
      <div class="card" style="width:340px; margin:auto;">
        <div style="display:flex; flex-direction:column; padding-bottom:12px;padding-top:0px;">
          Please select a target language:
          <select v-model="selectedLanguage" style="margin-top:4px">
            <option value="">Please select a language</option>
            <option v-for="language in languages" :key="language" :value="language">
              {{ language }}
            </option>
          </select>      
        </div>
        selectedImage: {{selectedImage}}
        selectedLanguage: {{selectedLanguage}}
        <button @click="captureImageAndGetResults()" style="width:100%">Get Results</button>            
        <audio ref="audioElement" controls style="width:100%; margin-top:12px"></audio>

      </div>
    </div>
  <canvas ref="canvas" style="display: none;"></canvas>   
  </main>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import axios from 'axios';


export default {
  onBeforeUnmount(){
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
    }    
  },
  mounted(){    
    this.initCamera()
  },
  methods: {
    getResults() {      
      console.log('!this.selectedImage', !this.selectedImage)
      console.log('this.selectedImage', this.selectedImage)
            
      console.log('!this.selectedLanguage', !this.selectedLanguage)
      if (!this.selectedImage || !this.selectedLanguage) {
        alert('Please select a language and upload an image');
        return;
      }
      let formData = new FormData();
      formData.append('language', this.selectedLanguage);
      formData.append('image', this.selectedImage);
      axios.post(this.backendURL, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(response => {
        console.log(response.data);
        let audioData = response.data.audio_bytes;
        let audioBlob = this.base64ToBlob(audioData, 'audio/mpeg');
        let audioUrl = URL.createObjectURL(audioBlob);
        this.$refs.audioElement.src = audioUrl;
      }).catch (error => {
        console.error(error);
      })
    },
    base64ToBlob(base64, mime) {
      mime = mime || '';
      var sliceSize = 1024;
      var byteChars = window.atob(base64);
      var byteArrays = [];

      for (var offset = 0, len = byteChars.length; offset < len; offset += sliceSize) {
        var slice = byteChars.slice(offset, offset + sliceSize);

        var byteNumbers = new Array(slice.length);
        for (var i = 0; i < slice.length; i++) {
          byteNumbers[i] = slice.charCodeAt(i);
        }

        var byteArray = new Uint8Array(byteNumbers);

        byteArrays.push(byteArray);
      }

      return new Blob(byteArrays, {type: mime});
    },    
    initCamera() {
      navigator.mediaDevices.getUserMedia({ video: true })
        .then(stream => {
          this.stream = stream;
          this.$refs.videoElement.srcObject = stream;
        })
        .catch(err => {
          console.error("Error accessing the camera:", err);
        });
    },
    captureImageAndGetResults() {
      console.log('capturing image')
      const canvas = this.$refs.canvas;
      const video = this.$refs.videoElement;
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      canvas.getContext('2d').drawImage(video, 0, 0);
      canvas.toBlob(blob => {
        console.log('capturing image')
        this.selectedImage = new File([blob], "image.jpg", { type: "image/jpeg" });
        this.getResults()
      }, 'image/jpeg');      
    }
  },
  data() {
    return {
      stream: null, 
      videoElement: null, 
      selectedImage: null,
      backendURL: 'http://localhost:5000/',
      languages: [        
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
  padding-bottom:12px;
  padding-left: 12px;
  padding-right: 12px;
}


</style>

<template>
  <div class="container">
    <h2>Upload Soccer Video</h2>
    <input type="file" @change="handleFileUpload" accept="video/*" />
    <button :disabled="!videoFile || loading" @click="uploadVideo">Upload</button>

    <p v-if="loading">Uploading and processing...</p>

    <div v-if="resultUrl">
      <h3>Processed Video:</h3>
      <button @click="downloadVideo">Download Processed Video</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: "UploadVideo",
  data() {
    return {
      videoFile: null,
      resultUrl: "",
      loading: false,
    }
  },
  methods: {
    handleFileUpload(event) {
      this.videoFile = event.target.files[0]
    },
    async uploadVideo() {
      if (!this.videoFile) return

      this.loading = true
      const formData = new FormData()
      formData.append("file", this.videoFile)

      try {
        const response = await axios.post("http://127.0.0.1:10000/upload", formData, {
          headers: {
            "Content-Type": "multipart/form-data"
          }
        })

        this.resultUrl = response.data.result_filepath
      } catch (error) {
        alert("Failed to upload or process video.")
        console.error(error)
      } finally {
        this.loading = false
      }
    },
    getFilename(url) {
      return url.substring(url.lastIndexOf('/') + 1)
    },
    async downloadVideo() {
      try {
        const response = await axios.get(this.resultUrl, {
          responseType: 'blob'
        })
        const blob = new Blob([response.data], { type: 'video/mp4' })
        const link = document.createElement('a')
        link.href = URL.createObjectURL(blob)
        link.download = this.getFilename(this.resultUrl)
        link.click()
        URL.revokeObjectURL(link.href)
      } catch (error) {
        console.error('❌ Failed to download video:', error)
        alert('Download gagal.')
      }
    }
  }
}
</script>


<style scoped>
.container {
  max-width: 700px;
  margin: auto;
  padding: 2rem;
}

.download-btn {
  display: inline-block;
  margin-top: 1rem;
  padding: 0.75rem 1.5rem;
  background-color: #4CAF50;
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-weight: bold;
}

.download-btn:hover {
  background-color: #45a049;
}
</style>

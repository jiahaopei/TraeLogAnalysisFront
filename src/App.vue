<template>
  <div class="container">
    <h1 class="title">日志智能分析系统</h1>
    
    <!-- 上传按钮 -->
    <div class="upload-section">
      <input 
        type="file" 
        ref="fileInput" 
        accept=".xlsx, .xls" 
        @change="handleFileUpload"
        style="display: none;"
      />
      <button class="upload-btn" @click="$refs.fileInput.click()">
        上传Excel文件
      </button>
      
      <!-- 上传成功提示 -->
      <div v-if="uploadedFiles.length > 0" class="upload-success">
        <span class="checkmark">✓</span>
        <span class="file-name">{{ uploadedFiles[uploadedFiles.length - 1].name }}</span>
      </div>
    </div>
    
    <!-- 下拉框和下载按钮 -->
    <div class="action-section">
      <select v-model="selectedFile" class="file-select">
        <option value="">请选择上传的文件</option>
        <option v-for="file in uploadedFiles" :key="file.id" :value="file">
          {{ file.name }}
        </option>
      </select>
      
      <button 
        class="download-btn" 
        @click="downloadResult"
        :disabled="!selectedFile"
      >
        下载结果
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      uploadedFiles: [],
      selectedFile: null,
      fileIdCounter: 0
    }
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0]
      if (file) {
        // 模拟上传成功
        const newFile = {
          id: this.fileIdCounter++,
          name: file.name,
          file: file
        }
        this.uploadedFiles.push(newFile)
        this.selectedFile = newFile
        
        // 清空文件输入，以便可以重复上传同一个文件
        event.target.value = ''
      }
    },
    downloadResult() {
      if (this.selectedFile) {
        // 模拟下载功能
        // 在实际应用中，这里应该调用后端API获取处理后的文件
        alert(`正在下载${this.selectedFile.name}的处理结果...`)
        
        // 模拟生成一个简单的txt文件进行下载
        const content = `这是${this.selectedFile.name}的处理结果示例`
        const blob = new Blob([content], { type: 'text/plain' })
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = `${this.selectedFile.name}_result.txt`
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
      }
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.title {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

.upload-section {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  gap: 15px;
}

.upload-btn {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.upload-btn:hover {
  background-color: #45a049;
}

.upload-success {
  display: flex;
  align-items: center;
  gap: 8px;
}

.checkmark {
  color: #4CAF50;
  font-size: 20px;
  font-weight: bold;
}

.file-name {
  color: #333;
  font-size: 14px;
}

.action-section {
  display: flex;
  gap: 15px;
  align-items: center;
}

.file-select {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  width: 300px;
}

.download-btn {
  padding: 10px 20px;
  background-color: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.download-btn:hover:not(:disabled) {
  background-color: #0b7dda;
}

.download-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}
</style>
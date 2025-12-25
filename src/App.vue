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
    
    <!-- 分析结果列表 -->
    <div class="result-section" v-if="uploadedFiles.length > 0">
      <table class="result-table">
        <thead>
          <tr>
            <th>文件名</th>
            <th>状态</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="file in uploadedFiles" :key="file.id">
            <td class="file-name-col">{{ file.name }}</td>
            <td class="status-col">
              <span :class="['status-tag', file.status]">
                {{ file.status === 'pending' ? '等待分析' : 
                   file.status === 'analyzing' ? '分析中' : 
                   file.status === 'completed' ? '分析完成' : '失败' }}
              </span>
            </td>
            <td class="action-col">
              <button 
                class="download-btn" 
                @click="downloadResult(file)"
                :disabled="file.status !== 'completed'"
              >
                下载结果
              </button>
              <button 
                class="error-btn" 
                @click="viewError(file)"
                :disabled="file.status !== 'failed'"
              >
                查看错误
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    
    <!-- 空状态提示 -->
    <div v-else class="empty-state">
      暂无上传文件
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      uploadedFiles: [],
      fileIdCounter: 0
    }
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0]
      if (file) {
        // 模拟上传成功，初始状态为等待分析
        const newFile = {
          id: this.fileIdCounter++,
          name: file.name,
          file: file,
          status: 'pending', // pending: 等待分析, analyzing: 分析中, completed: 分析完成, failed: 失败
          error: null
        }
        this.uploadedFiles.push(newFile)
        
        // 模拟分析过程
        this.simulateAnalysis(newFile)
        
        // 清空文件输入，以便可以重复上传同一个文件
        event.target.value = ''
      }
    },
    
    // 模拟分析过程
    simulateAnalysis(file) {
      // 1秒后状态变为分析中
      setTimeout(() => {
        file.status = 'analyzing'
        
        // 3秒后随机变为分析完成或失败
        setTimeout(() => {
          // 80%的概率成功，20%的概率失败
          if (Math.random() > 0.2) {
            file.status = 'completed'
          } else {
            file.status = 'failed'
            file.error = '分析过程中发生错误，无法完成处理。'
          }
        }, 3000)
      }, 1000)
    },
    
    downloadResult(file) {
      if (file.status === 'completed') {
        // 模拟下载功能
        // 在实际应用中，这里应该调用后端API获取处理后的文件
        alert(`正在下载${file.name}的处理结果...`)
        
        // 模拟生成一个简单的txt文件进行下载
        const content = `这是${file.name}的处理结果示例`
        const blob = new Blob([content], { type: 'text/plain' })
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = `${file.name}_result.txt`
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
      }
    },
    
    viewError(file) {
      if (file.status === 'failed') {
        alert(`文件 ${file.name} 的错误信息：${file.error}`)
      }
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 900px;
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

.result-section {
  margin-top: 30px;
}

.result-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.result-table th,
.result-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.result-table th {
  background-color: #f5f5f5;
  font-weight: bold;
  color: #333;
}

.result-table tr:hover {
  background-color: #f9f9f9;
}

.file-name-col {
  max-width: 300px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.status-col {
  width: 150px;
}

.status-tag {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
  color: #fff;
}

.status-tag.pending {
  background-color: #ffc107;
}

.status-tag.analyzing {
  background-color: #2196F3;
}

.status-tag.completed {
  background-color: #4CAF50;
}

.status-tag.failed {
  background-color: #f44336;
}

.action-col {
  width: 200px;
  display: flex;
  gap: 10px;
}

.download-btn,
.error-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
}

.download-btn {
  background-color: #2196F3;
  color: white;
}

.download-btn:hover:not(:disabled) {
  background-color: #0b7dda;
}

.download-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.error-btn {
  background-color: #f44336;
  color: white;
}

.error-btn:hover:not(:disabled) {
  background-color: #d32f2f;
}

.error-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.empty-state {
  margin-top: 30px;
  text-align: center;
  color: #666;
  padding: 40px 0;
  background-color: #f5f5f5;
  border-radius: 4px;
}
</style>
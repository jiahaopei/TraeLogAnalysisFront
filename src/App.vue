<template>
  <div class="container">
    <h1 class="title">日志智能分析系统</h1>
    
    <!-- 错误提示 -->
    <div v-if="errorVisible" :class="['error-message', { 'is-success': !isErrorMessage }]">
      <span class="error-text">{{ errorMessage }}</span>
      <button class="close-btn" @click="closeError">×</button>
    </div>
    
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
      <div v-if="uploadSuccess" class="upload-success">
        <span class="checkmark">✓</span>
        <span class="file-name">{{ uploadSuccess }}</span>
      </div>
    </div>
    
    <!-- 分析结果列表 -->
    <div class="result-section" v-if="totalElements > 0">
      <table class="result-table">
        <thead>
          <tr>
            <th>文件名</th>
            <th>状态</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="file in files" :key="file.id">
            <td class="file-name-col">{{ file.fileName }}</td>
            <td class="status-col">
              <span :class="['status-tag', getStatusClass(file.status)]">
                {{ getStatusText(file.status) }}
              </span>
            </td>
            <td class="action-col">
              <button 
                class="analyze-btn" 
                @click="startAnalysis(file)"
                :disabled="file.status === 'ANALYZING' || file.status === 'COMPLETED'"
              >
                开始分析
              </button>
              <button 
                class="download-btn" 
                @click="downloadResult(file)"
                :disabled="file.status !== 'COMPLETED'"
              >
                下载结果
              </button>
              <button 
                class="error-btn" 
                @click="viewError(file)"
                :disabled="file.status !== 'FAILED'"
              >
                查看错误
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      
      <!-- 分页控件 -->
      <div class="pagination">
        <button 
          @click="changePage(1)" 
          :disabled="currentPage === 1"
          class="page-btn"
        >
          首页
        </button>
        <button 
          @click="changePage(currentPage - 1)" 
          :disabled="currentPage === 1"
          class="page-btn"
        >
          上一页
        </button>
        <span class="page-info">
          第 {{ currentPage }} 页 / 共 {{ totalPages }} 页
        </span>
        <button 
          @click="changePage(currentPage + 1)" 
          :disabled="currentPage === totalPages"
          class="page-btn"
        >
          下一页
        </button>
        <button 
          @click="changePage(totalPages)" 
          :disabled="currentPage === totalPages"
          class="page-btn"
        >
          末页
        </button>
        <select v-model="pageSize" @change="changePageSize" class="page-size-select">
          <option :value="5">5条/页</option>
          <option :value="10">10条/页</option>
          <option :value="20">20条/页</option>
          <option :value="50">50条/页</option>
        </select>
      </div>
    </div>
    
    <!-- 空状态提示 -->
    <div v-else class="empty-state">
      暂无上传文件
    </div>
    
    <!-- 错误详情模态框 -->
    <div v-if="detailModalVisible" class="modal-overlay" @click="closeDetailModal">
      <div class="modal" @click.stop>
        <div class="modal-header">
          <h3>错误详情</h3>
          <button class="modal-close" @click="closeDetailModal">×</button>
        </div>
        <div class="modal-content">
          <div class="modal-field">
            <label>文件名称：</label>
            <span>{{ detailFileName }}</span>
          </div>
          <div class="modal-field">
            <label>错误信息：</label>
            <textarea readonly class="error-textarea">{{ detailErrorMessage }}</textarea>
          </div>
        </div>
        <div class="modal-footer">
          <button class="modal-btn" @click="closeDetailModal">确定</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { API_BASE_URL } from './config'

export default {
  name: 'App',
  data() {
    return {
      files: [], // 当前页的文件列表
      totalElements: 0, // 总记录数
      totalPages: 0, // 总页数
      currentPage: 1, // 当前页码
      pageSize: 10, // 每页大小
      uploadSuccess: '', // 上传成功提示
      loading: false, // 加载状态
      errorMessage: '', // 错误提示信息
      errorVisible: false, // 错误提示是否显示
      isErrorMessage: true, // 是否是错误信息，false为成功信息
      // 错误详情模态框
      detailModalVisible: false, // 模态框是否显示
      detailFileName: '', // 错误文件名称
      detailErrorMessage: '' // 详细错误信息
    }
  },
  mounted() {
    // 页面加载时获取文件列表
    this.getFiles()
    // 每5秒刷新一次列表，更新状态
    this.refreshInterval = setInterval(() => {
      this.getFiles()
    }, 60000)
  },
  beforeUnmount() {
    // 组件销毁前清除定时器
    if (this.refreshInterval) {
      clearInterval(this.refreshInterval)
    }
  },
  methods: {
    // 显示错误或成功信息
    showError(message, isError = true) {
      this.errorMessage = message
      this.errorVisible = true
      this.isErrorMessage = isError
      // 5秒后自动关闭提示
      setTimeout(() => {
        this.closeError()
      }, 5000)
    },
    
    // 关闭错误信息
    closeError() {
      this.errorVisible = false
      this.errorMessage = ''
    },
    
    // 获取文件列表
    async getFiles() {
      try {
        this.loading = true
        // 关闭之前的错误提示
        this.closeError()
        
        const response = await axios.get(`${API_BASE_URL}/api/files/page`, {
          params: {
            page: this.currentPage - 1, // 后端是从0开始的页码
            size: this.pageSize
          }
        })
        
        const pageData = response.data
        // 空值检查，确保分页数据安全处理
        this.files = Array.isArray(pageData.content) ? pageData.content : []
        this.totalElements = Number(pageData.totalElements) || 0
        this.totalPages = Number(pageData.totalPages) || 0
        
        // 如果当前页没有数据且不是第一页，则跳转到前一页
        if (this.files.length === 0 && this.currentPage > 1) {
          this.currentPage--
          this.getFiles()
        }
      } catch (error) {
        console.error('获取文件列表失败:', error)
        // 发生错误时重置列表数据，避免页面显示异常
        this.files = []
        this.totalElements = 0
        this.totalPages = 0
        
        // 显示友好的错误提示
        if (error.response) {
          // 服务器返回错误响应
          this.showError(`获取文件列表失败：${error.response.status} ${error.response.statusText}`)
        } else if (error.request) {
          // 请求已发送但没有收到响应
          this.showError('获取文件列表失败：无法连接到服务器，请检查后端服务是否运行')
        } else {
          // 请求配置错误
          this.showError(`获取文件列表失败：${error.message}`)
        }
      } finally {
        this.loading = false
      }
    },
    
    // 上传文件
    async handleFileUpload(event) {
      const file = event.target.files[0]
      if (file) {
        try {
          this.loading = true
          this.closeError()
          const formData = new FormData()
          formData.append('file', file)
          
          const response = await axios.post(`${API_BASE_URL}/api/files/upload`, formData, {
            headers: {
              'Content-Type': 'multipart/form-data'
            }
          })
          
          // 显示上传成功提示
          this.uploadSuccess = response.data.fileName
          // 3秒后清除提示
          setTimeout(() => {
            this.uploadSuccess = ''
          }, 3000)
          
          // 刷新文件列表
          this.getFiles()
          
          // 清空文件输入，以便可以重复上传同一个文件
          event.target.value = ''
        } catch (error) {
          console.error('上传文件失败:', error)
          if (error.response) {
            this.showError(`上传文件失败：${error.response.status} ${error.response.statusText}`)
          } else if (error.request) {
            this.showError('上传文件失败：无法连接到服务器，请检查后端服务是否运行')
          } else {
            this.showError(`上传文件失败：${error.message}`)
          }
        } finally {
          this.loading = false
        }
      }
    },
    
    // 开始分析
    async startAnalysis(file) {
      try {
        this.loading = true
        this.closeError()
        await axios.post(`${API_BASE_URL}/api/analysis/start/${file.id}`)
        // 显示成功提示
        this.errorMessage = '分析已开始'
        this.errorVisible = true
        this.isErrorMessage = false
        // 5秒后自动关闭提示
        setTimeout(() => {
          this.closeError()
        }, 5000)
        // 刷新文件列表
        this.getFiles()
      } catch (error) {
        console.error('开始分析失败:', error)
        if (error.response) {
          this.showError(`开始分析失败：${error.response.status} ${error.response.statusText}`)
        } else if (error.request) {
          this.showError('开始分析失败：无法连接到服务器，请检查后端服务是否运行')
        } else {
          this.showError(`开始分析失败：${error.message}`)
        }
      } finally {
        this.loading = false
      }
    },
    
    // 下载结果
    async downloadResult(file) {
      try {
        this.loading = true
        this.closeError()
        const response = await axios.get(`${API_BASE_URL}/api/results/export/${file.id}`, {
          responseType: 'blob' // 重要：设置响应类型为blob
        })
        
        // 从响应头中获取文件名
        const contentDisposition = response.headers['content-disposition']
        let fileName = `${file.fileName}_result.xlsx`
        if (contentDisposition) {
          const match = contentDisposition.match(/filename="?([^"]+)"?/)
          if (match && match[1]) {
            fileName = match[1]
          }
        }
        
        // 创建下载链接
        const blob = new Blob([response.data])
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = fileName
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
      } catch (error) {
        console.error('下载结果失败:', error)
        if (error.response) {
          this.showError(`下载结果失败：${error.response.status} ${error.response.statusText}`)
        } else if (error.request) {
          this.showError('下载结果失败：无法连接到服务器，请检查后端服务是否运行')
        } else {
          this.showError(`下载结果失败：${error.message}`)
        }
      } finally {
        this.loading = false
      }
    },
    
    // 获取状态中文文本
    getStatusText(status) {
      const statusMap = {
        'UPLOADED': '等待分析',
        'ANALYZING': '分析中',
        'COMPLETED': '分析完成',
        'FAILED': '失败'
      }
      return statusMap[status] || status
    },
    
    // 获取状态对应的CSS类
    getStatusClass(status) {
      const classMap = {
        'UPLOADED': 'pending',
        'ANALYZING': 'analyzing',
        'COMPLETED': 'completed',
        'FAILED': 'failed'
      }
      return classMap[status] || ''
    },
    
    // 查看错误
    viewError(file) {
      // 显示错误详情模态框
      this.detailFileName = file.fileName
      this.detailErrorMessage = file.errorMessage || '未知错误'
      this.detailModalVisible = true
    },
    
    // 关闭错误详情模态框
    closeDetailModal() {
      this.detailModalVisible = false
      this.detailFileName = ''
      this.detailErrorMessage = ''
    },
    
    // 切换页码
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page
        this.getFiles()
      }
    },
    
    // 切换每页大小
    changePageSize() {
      this.currentPage = 1 // 重置到第一页
      this.getFiles()
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

.analyze-btn,
.download-btn,
.error-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
}

.analyze-btn {
  background-color: #ff9800;
  color: white;
}

.analyze-btn:hover:not(:disabled) {
  background-color: #f57c00;
}

.analyze-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
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

/* 错误提示样式 */
.error-message {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  margin-bottom: 20px;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
  background-color: #fef2f2;
  color: #dc2626;
  border: 1px solid #fecaca;
  transition: all 0.3s ease;
}

.error-message:not(.is-success) {
  background-color: #fef2f2;
  color: #dc2626;
  border: 1px solid #fecaca;
}

.error-message.is-success {
  background-color: #f0fdf4;
  color: #16a34a;
  border: 1px solid #bbf7d0;
}

.error-text {
  flex: 1;
}

.close-btn {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  color: inherit;
  padding: 0;
  margin-left: 12px;
  line-height: 1;
}

.close-btn:hover {
  opacity: 0.7;
}

/* 分页样式 */
.pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
  gap: 10px;
}

.page-btn {
  padding: 8px 16px;
  border: 1px solid #ddd;
  background-color: #fff;
  color: #333;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.page-btn:hover:not(:disabled) {
  background-color: #f5f5f5;
  border-color: #2196F3;
  color: #2196F3;
}

.page-btn:disabled {
  background-color: #f5f5f5;
  border-color: #ddd;
  color: #999;
  cursor: not-allowed;
}

.page-info {
  font-size: 14px;
  color: #666;
  margin: 0 10px;
}

.page-size-select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background-color: #fff;
  color: #333;
  cursor: pointer;
}

.page-size-select:focus {
  outline: none;
  border-color: #2196F3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.1);
}

/* 模态框样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

.modal {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  width: 90%;
  max-width: 600px;
  max-height: 80vh;
  overflow-y: auto;
  animation: slideIn 0.3s ease;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid #e5e7eb;
  background-color: #f9fafb;
  border-radius: 8px 8px 0 0;
}

.modal-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #111827;
}

.modal-close {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #6b7280;
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.2s;
}

.modal-close:hover {
  background-color: #ef4444;
  color: #fff;
}

.modal-content {
  padding: 20px;
}

.modal-field {
  margin-bottom: 20px;
}

.modal-field label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #374151;
  font-size: 14px;
}

.modal-field span {
  display: block;
  padding: 8px 12px;
  background-color: #f3f4f6;
  border: 1px solid #e5e7eb;
  border-radius: 4px;
  color: #111827;
  font-size: 14px;
}

.error-textarea {
  width: 100%;
  min-height: 150px;
  padding: 12px;
  border: 1px solid #e5e7eb;
  border-radius: 4px;
  background-color: #f3f4f6;
  color: #111827;
  font-size: 14px;
  font-family: inherit;
  resize: vertical;
  line-height: 1.5;
}

.error-textarea:focus {
  outline: none;
  border-color: #2196F3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.1);
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  padding: 16px 20px;
  border-top: 1px solid #e5e7eb;
  background-color: #f9fafb;
  border-radius: 0 0 8px 8px;
}

.modal-btn {
  padding: 8px 16px;
  background-color: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s;
}

.modal-btn:hover {
  background-color: #0b7dda;
}

/* 动画效果 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
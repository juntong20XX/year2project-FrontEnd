<template>
  <div class="item-list">
    <div class="header">
      <h1>Modules</h1>
      <button class="logout-btn" @click="logout">Logout</button>
    </div>
    <table>
      <thead>
      <tr>
        <th>Name</th>
        <th>Sensor</th>
        <th>Action</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="item in items" :key="item.key">
        <td>{{ item.key }}</td>
        <td>
          <div class="degree-container">
            <input
                type="number"
                v-model.number="item.value"
                min="45"
                max="135"
                @focus="markFocusedInput(item.key)"
                @blur="unmarkFocusedInput"
            >
            <span v-if="getRealValue(item.key) !== item.value" class="real-value">
              Actual Value: {{ getRealValue(item.key) }}
            </span>
          </div>
        </td>
        <td>
          <div class="action-buttons">
            <button @click="updateItem(item)">Update</button>
            <button 
              v-if="getRealValue(item.key) !== item.value"
              class="restore-btn"
              @click="restoreValue(item)"
            >Recover</button>
          </div>
        </td>
      </tr>
      </tbody>
    </table>

    <!-- Success Modal -->
    <div v-if="showSuccessModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showSuccessModal = false">&times;</span>
        <h3>Success</h3>
        <p>Item updated successfully</p>
        <button @click="showSuccessModal = false">Close</button>
      </div>
    </div>

    <!-- Removed Item Modal -->
    <div v-if="showRemovedItemModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showRemovedItemModal = false">&times;</span>
        <h3>Item Removed</h3>
        <p>The item you were editing no longer exists in the backend.</p>
        <button @click="showRemovedItemModal = false">Close</button>
      </div>
    </div>

    <!-- Range Error Modal -->
    <div v-if="showRangeErrorModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showRangeErrorModal = false">&times;</span>
        <h3>Invalid Range</h3>
        <p>Value must be between 45 and 135</p>
        <button class="error-btn" @click="showRangeErrorModal = false">Close</button>
      </div>
    </div>

    <!-- Backend Error Modal -->
    <div v-if="showBackendErrorModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showBackendErrorModal = false">&times;</span>
        <h3>Connection Error</h3>
        <p>Unable to connect to backend server</p>
        <div class="modal-buttons">
          <button @click="retryConnection">Retry Connection</button>
          <button class="error-btn" @click="logout">Logout</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'ItemList',
  data() {
    return {
      items: [],
      previousItems: [],
      backendUrl: 'http://localhost:8000', // 默认后端地址
      showSuccessModal: false,
      showRemovedItemModal: false,
      showRangeErrorModal: false,
      showBackendErrorModal: false,
      updateInterval: null,
      focusedInputKey: null
    }
  },
  mounted() {
    // 从 cookie 读取后端地址
    const cookies = document.cookie.split(';')
    for (let cookie of cookies) {
      const [name, value] = cookie.trim().split('=')
      if (name === 'backendUrl') {
        this.backendUrl = value
        break
      }
    }
    this.fetchItems()
    // 每秒更新数据
    this.startUpdateInterval()
  },
  beforeDestroy() {
    // 组件销毁前清除定时器
    this.stopUpdateInterval()
  },
  methods: {
    startUpdateInterval() {
      this.updateInterval = setInterval(() => {
        this.fetchItems()
      }, 1000)
    },
    stopUpdateInterval() {
      if (this.updateInterval) {
        clearInterval(this.updateInterval)
        this.updateInterval = null
      }
    },
    async fetchItems() {
      try {
        const response = await axios.get(`${this.backendUrl}/items`)
        const newItems = response.data

        // 如果数据没有变化，不更新
        if (JSON.stringify(this.previousItems) === JSON.stringify(newItems)) {
          return
        }

        // 处理用户正在编辑的输入框
        if (this.focusedInputKey) {
          const focusedItem = newItems.find(item => item.key === this.focusedInputKey)
          if (!focusedItem) {
            // 如果正在编辑的项目已从后端移除
            this.showRemovedItemModal = true
            this.focusedInputKey = null
          } else {
            // 保留用户正在编辑的输入框的值
            const currentItem = this.items.find(item => item.key === this.focusedInputKey)
            focusedItem.value = currentItem.value
          }
        }

        this.previousItems = this.items
        this.items = newItems
        
        // 如果成功连接，确保错误模态框关闭
        this.showBackendErrorModal = false
      } catch (error) {
        console.error('Error fetching items:', error)
        if (error.response?.status === 404 || !error.response) {
          this.stopUpdateInterval()
          this.showBackendErrorModal = true
          // 清空列表
          this.items = []
          this.previousItems = []
        } else {
          this.handleError(error)
        }
      }
    },
    async updateItem(item) {
      try {
        if (item.value < 45 || item.value > 135) {
          this.showRangeErrorModal = true
          return
        }

        await axios.put(`${this.backendUrl}/items`, {
          key: item.key,
          value: item.value
        })

        this.showSuccessModal = true
      } catch (error) {
        console.error('Error updating item:', error)
        if (error.response?.status === 404 || !error.response) {
          this.stopUpdateInterval()
          this.showBackendErrorModal = true
          // 清空列表
          this.items = []
          this.previousItems = []
        } else {
          this.handleError(error)
        }
      }
    },
    markFocusedInput(key) {
      this.focusedInputKey = key
    },
    unmarkFocusedInput() {
      this.focusedInputKey = null
    },
    handleError(error) {
      const errorMessage = error.response?.data?.detail || 'An unexpected error occurred'
      alert(errorMessage)
    },
    logout() {
      this.stopUpdateInterval()
      this.$emit('logout')
    },
    getRealValue(key) {
      const item = this.previousItems.find(item => item.key === key)
      return item ? item.value : null
    },
    restoreValue(item) {
      const realValue = this.getRealValue(item.key)
      if (realValue !== null) {
        item.value = realValue
      }
    },
    retryConnection() {
      this.showBackendErrorModal = false
      this.fetchItems()
      this.startUpdateInterval()
    }
  }
}
</script>

<style scoped>
.item-list {
  max-width: 600px;
  margin: 0 auto;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.logout-btn {
  background-color: #f44336;
  width: auto;
  padding: 8px 16px;
}

.logout-btn:hover {
  background-color: #d32f2f;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

input {
  width: 100%;
  padding: 5px;
  box-sizing: border-box;
}

button {
  width: 100%;
  padding: 5px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  margin-bottom: 4px;
}

button:hover {
  background-color: #45a049;
}

.error-btn {
  background-color: #f44336;
}

.error-btn:hover {
  background-color: #d32f2f;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  position: relative;
  width: 300px;
  text-align: center;
}

.close {
  position: absolute;
  right: 10px;
  top: 5px;
  font-size: 20px;
  cursor: pointer;
}

.restore-btn {
  background-color: #2196F3;
}

.restore-btn:hover {
  background-color: #1976D2;
}

.action-buttons {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
</style>
<template>
  <div class="login-container">
    <h2>TOTP Login</h2>
    <div class="login-form">
      <input
          v-model="username"
          placeholder="Username"
          @keyup.enter="login"
          :disabled="isLoading"
      />
      <input
          v-model="totpCode"
          placeholder="TOTP Code"
          @keyup.enter="login"
          :disabled="isLoading"
      />
      <input
          v-model="backendUrl"
          placeholder="Backend URL"
          @keyup.enter="login"
          :disabled="isLoading"
      />
      <button @click="login" :disabled="isLoading">
        {{ isLoading ? 'Logging in...' : 'Login' }}
      </button>

      <div class="register-info">
        <p>Need an account?</p>
        <button @click="showRegisterInfo" class="register-btn">
          Request Registration
        </button>
      </div>
    </div>

    <!-- 注册信息模态框 -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showModal = false">&times;</span>
        <h3>Account Registration</h3>
        <p>To request an account, please contact our administrator at:</p>
        <p><strong>admin@example.com</strong></p>
        <button @click="showModal = false">Close</button>
      </div>
    </div>

    <!-- 登录失败模态框 -->
    <div v-if="showLoginErrorModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showLoginErrorModal = false">&times;</span>
        <h3>Login Failed</h3>
        <p>{{ loginErrorMessage }}</p>
        <button @click="showLoginErrorModal = false">Close</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Login',
  data() {
    return {
      username: '',
      totpCode: '',
      backendUrl: 'http://localhost:8000', // 默认后端URL
      showModal: false,
      showLoginErrorModal: false,
      loginErrorMessage: '',
      isLoading: false
    }
  },
  methods: {
    async login() {
      // 验证输入
      if (!this.username || !this.totpCode || !this.backendUrl) {
        this.loginErrorMessage = 'Please enter username, TOTP code and backend URL'
        this.showLoginErrorModal = true
        return
      }

      this.isLoading = true
      try {
        const response = await axios.post(`${this.backendUrl}/login`, {
          username: this.username,
          totp_code: this.totpCode
        }, {
          timeout: 5000 // 设置后端 URL 5 秒超时
        })

        // 将 backendUrl 写入 cookie，有效期 7 天
        document.cookie = `backendUrl=${this.backendUrl};max-age=604800;path=/`
        
        // 登录成功
        this.$emit('login-success')
      } catch (error) {
        console.error('Login error:', error)
        if (error.code === 'ECONNABORTED') {
          this.loginErrorMessage = 'Backend URL timeout. Please check it.'
        } else {
          this.loginErrorMessage = error.response?.data?.detail || 'Login failed'
        }
        this.showLoginErrorModal = true
      } finally {
        this.isLoading = false
      }
    },
    showRegisterInfo() {
      this.showModal = true
    }
  }
}
</script>

<style scoped>
.login-container {
  max-width: 400px;
  margin: 100px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  text-align: center;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

input {
  width: 100%;
  padding: 10px;
  margin: 5px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

input:disabled {
  background-color: #f5f5f5;
  cursor: not-allowed;
}

button {
  padding: 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #45a049;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.register-info {
  margin-top: 15px;
}

.register-btn {
  background-color: #2196F3;
}

.register-btn:hover {
  background-color: #1976D2;
}

/* 模态框样式 */
.modal {
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: #fefefe;
  padding: 20px;
  border: 1px solid #888;
  width: 80%;
  max-width: 500px;
  border-radius: 8px;
  text-align: left;
  position: relative;
}

.close {
  color: #aaa;
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
}

.close:hover {
  color: black;
}
</style>
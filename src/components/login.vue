<template>
  <div class="login-container">
    <h2>TOTP Login</h2>
    <div class="login-form">
      <input
          v-model="username"
          placeholder="Username"
          @keyup.enter="login"
      />
      <input
          v-model="totpCode"
          placeholder="TOTP Code"
          @keyup.enter="login"
      />
      <button @click="login">Login</button>

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
      showModal: false
    }
  },
  methods: {
    async login() {
      // 验证输入
      if (!this.username || !this.totpCode) {
        alert('Please enter username and TOTP code')
        return
      }

      try {
        const response = await axios.post('http://localhost:8000/login', {
          username: this.username,
          totp_code: this.totpCode
        })

        // 登录成功
        this.$emit('login-success')
        alert('Login successful')
      } catch (error) {
        console.error('Login error:', error)
        alert(error.response?.data?.detail || 'Login failed')
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
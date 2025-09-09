<template>
  <div class="login-wrapper" :class="resetToken ? 'reset-bg' : 'main-gradient'">
    <div class="login-card" v-if="isQuxAuth">
      
      <!-- Tabs -->
      <div class="tabs" v-if="!resetToken">
        <a :class="{active: tab === 'login'}" @click="setTab('login')">Login</a>
        <a :class="{active: tab === 'signup'}" @click="setTab('signup')" v-if="allowSignUp">Sign Up</a>
      </div>

      <!-- Login Form -->
      <div v-if="tab === 'login'" class="form-section">
        <h2>Welcome Back 👋</h2>
        <div class="input-group">
          <label>Email</label>
          <input type="text" placeholder="Your email" v-model="email" />
        </div>
        <div class="input-group">
          <label>Password</label>
          <input type="password" placeholder="Your password" v-model="password" @keyup.enter="login" />
        </div>
        <button class="btn-primary" @click="login">Login</button>

        <!-- Forgot Password always visible -->
        <a class="link" @click="requestPasswordReset">Forgot Password?</a>

        <p v-if="errorMessage" class="error-msg">{{ errorMessage }}</p>
      </div>

      <!-- Signup Form -->
      <div v-if="tab === 'signup'" class="form-section">
        <h2>Create Account 🚀</h2>
        <div class="input-group">
          <label>Email</label>
          <input type="text" placeholder="Your email" v-model="email" />
        </div>
        <div class="input-group">
          <label>Password</label>
          <input type="password" placeholder="Choose a password" v-model="password" @keyup.enter="signup" />
        </div>
        <div class="checkbox-row">
          <CheckBox v-model="tos" label="" />
          <span @click="tos=true">
            I accept the <a href="#/tos.html" target="_blank">terms of service</a>
          </span>
        </div>
        <button class="btn-primary" @click="signup">Sign Up</button>
        <p v-if="errorMessage" class="error-msg">{{ errorMessage }}</p>
      </div>

      <!-- Reset Password -->
      <div v-if="tab === 'reset'" class="form-section">
        <h2>Reset Password 🔑</h2>
        <div class="input-group">
          <label>Email</label>
          <input type="text" placeholder="Your email" v-model="email" />
        </div>
        <div class="input-group">
          <label>New Password</label>
          <input type="password" placeholder="Enter new password" v-model="password" />
        </div>
        <button class="btn-danger" @click="resetPassword">Set New Password</button>
        <p v-if="errorMessage" class="error-msg">{{ errorMessage }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Backgrounds */
.main-gradient {
  min-height: 100vh;
  background: linear-gradient(135deg, #4f46e5, #9333ea);
  display: flex;
  align-items: center;
  justify-content: center;
}
.reset-bg {
  min-height: 100vh;
  background: linear-gradient(135deg, #f43f5e, #f97316);
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Card */
.login-card {
  background: #fff;
  padding: 2rem;
  border-radius: 1rem;
  box-shadow: 0 6px 20px rgba(0,0,0,0.15);
  width: 100%;
  max-width: 400px;
}

/* Tabs */
.tabs {
  display: flex;
  justify-content: center;
  margin-bottom: 1.2rem;
}
.tabs a {
  flex: 1;
  text-align: center;
  padding: 0.7rem;
  border-radius: 999px;
  font-weight: 600;
  cursor: pointer;
  background: #f3f4f6;
  margin: 0 0.25rem;
  transition: all 0.3s;
}
.tabs a.active {
  background: #4f46e5;
  color: #fff;
}

/* Form */
.form-section h2 {
  text-align: center;
  margin-bottom: 1.2rem;
  color: #111827;
}
.input-group {
  margin-bottom: 1rem;
}
.input-group label {
  font-size: 0.85rem;
  color: #374151;
  margin-bottom: 0.25rem;
  display: block;
}
.input-group input {
  width: 100%;
  padding: 0.7rem;
  border-radius: 0.5rem;
  border: 1px solid #d1d5db;
  outline: none;
  transition: border 0.3s;
}
.input-group input:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 0 2px rgba(79,70,229,0.2);
}

/* Buttons */
.btn-primary {
  width: 100%;
  padding: 0.8rem;
  background: #4f46e5;
  color: #fff;
  border: none;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  margin-top: 0.5rem;
  transition: background 0.3s;
}
.btn-primary:hover {
  background: #4338ca;
}
.btn-danger {
  width: 100%;
  padding: 0.8rem;
  background: #ef4444;
  color: #fff;
  border: none;
  border-radius: 0.5rem;
  font-weight: 600;
  cursor: pointer;
  margin-top: 0.5rem;
}
.btn-danger:hover {
  background: #dc2626;
}

/* Links + Error */
.link {
  display: block;
  text-align: right;
  margin-top: 0.75rem;
  font-size: 0.85rem;
  color: #4f46e5;
  cursor: pointer;
}
.link:hover {
  text-decoration: underline;
}
.error-msg {
  color: #ef4444;
  font-size: 0.85rem;
  margin-top: 0.75rem;
  text-align: center;
}

/* Checkbox */
.checkbox-row {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.85rem;
  margin-bottom: 1rem;
}
</style>

<script>
import Services from 'services/Services'
import Logger from 'common/Logger'
import CheckBox from '../common/CheckBox.vue'

export default {
  name: "LoginPage",
  props: ['user'],
  data() {
    return {
      hasLoginError: false,
      resetToken: false,
      email: '',
      password: '',
      tos: false,
      errorMessage: '',
      signupInProgress: false,
      tab: 'login',
      config: {}
    }
  },
  computed: {
    isQuxAuth() {
      return Services.getConfig().auth !== 'keycloak'
    },
    allowSignUp() {
      return this.config && this.config.user && this.config.user.allowSignUp === true
    }
  },
  watch: {
    user(v) {
      this.logger.log(6, 'watch', 'user >> ' + v.email)
      this.user = v
    }
  },
  components: { CheckBox },
  methods: {
    setTab(tab) {
      this.tab = tab
      this.errorMessage = ''
    },
    async resetPassword() {
      this.logger.info('resetPassword', 'enter ', this.email)
      if (this.email.length < 2) {
        this.errorMessage = "Please enter your email"
        return
      }
      if (this.password.length < 6) {
        this.errorMessage = "Password too short"
        return
      }
      if (this.resetToken.length < 6) {
        this.errorMessage = "Token is wrong"
        return
      }
      let result = await Services.getUserService().reset2(this.email, this.password, this.resetToken)
      if (result.type === 'error') {
        this.errorMessage = 'Something went wrong'
      } else {
        this.errorMessage = ''
        this.resetToken = ''
        this.tab = 'login'
        this.$router.push('/')
      }
    },
    async requestPasswordReset() {
      this.logger.info('requestPasswordReset', 'enter ', this.email)
      await Services.getUserService().reset(this.email)
      this.errorMessage = 'Check your mail.'
    },
    async login() {
      this.logger.info('login', 'enter ', this.email)
      var result = await Services.getUserService().login({
        email: this.email,
        password: this.password
      })
      if (result.type == "error") {
        this.$root.$emit("Error", "Wrong login credentials")
        this.errorMessage = "Login is wrong"
        this.hasLoginError = true
      } else {
        this.$emit('login', result)
        this.$root.$emit('UserLogin', result)
        this.hasLoginError = false
      }
    },
    async signup() {
      this.logger.info('signup', 'enter ', this.email)
      if (this.signupInProgress) return
      if (this.password.length < 6) {
        this.errorMessage = "Password too short"
        return
      }
      if (this.tos !== true) {
        this.errorMessage = "Please accept terms of service"
        return
      }
      this.signupInProgress = true
      var result = await Services.getUserService().signup({
        email: this.email,
        password: this.password,
        tos: this.tos
      })
      this.signupInProgress = false
      if (result.type == "error") {
        if (result.errors.indexOf("user.create.domain") >= 0) {
          this.errorMessage = "Not the correct domain"
        } else if (result.errors.indexOf("user.create.nosignup") >= 0) {
          this.errorMessage = "No sign-ups allowed"
        } else if (result.errors.indexOf("user.email.not.unique") >= 0) {
          this.errorMessage = "Email is taken"
        } else {
          this.errorMessage = "Password too short"
        }
      } else {
        let user = await Services.getUserService().login({
          email: this.email,
          password: this.password
        })
        this.$emit('login', user)
        this.$root.$emit('UserLogin', user)
      }
    }
  },
  async mounted() {
    this.logger = new Logger('LoginPage')
    this.resetToken = this.$route.query.id
    if (this.resetToken && this.resetToken.length > 2) {
      this.tab = 'reset'
    }
    this.config = Services.getConfig()
  }
}
</script>

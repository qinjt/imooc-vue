<template>
  <header class="header">
    <div class="navbar">
      <div class="navbar-left-container">
        <a href="/">
          <img class="navbar-brand-logo" src="static/logo.png"></a>
      </div>
      <div class="navbar-right-container" style="display: flex;">
        <div class="navbar-menu-container">
          <!--<a href="/" class="navbar-link">我的账户</a>-->
          <span class="navbar-link" v-if="nickName">{{nickName}}</span>
          <a href="javascript:void(0)" class="navbar-link" v-if="!nickName" @click="loginModalFlag=true">Login</a>
          <a href="javascript:void(0)" class="navbar-link" v-if="nickName" @click="logOut">Logout</a>
          <div class="navbar-cart-container">
            <span class="navbar-cart-count" v-if="cartCount" v-text="cartCount"></span>
            <a class="navbar-link navbar-cart-link" href="/#/cart">
              <svg class="navbar-cart-logo">
                <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-cart"></use>
              </svg>
            </a>
          </div>
        </div>
      </div>
    </div>
    <div class="md-modal modal-msg md-modal-transition" :class="{'md-show':loginModalFlag}">
      <div class="md-modal-inner">
        <div class="md-top">
          <div class="md-title">Login in</div>
          <button class="md-close" @click="loginModalFlag=false">CLose</button>
        </div>
        <div class="md-content">
          <div class="confirm-tips">
            <div class="error-wrap">
              <span class="error error-show" v-show="errorTip">用户名或者密码错误</span>
            </div>
            <ul>
              <li class="regi_form_input">
                <i class="icon IconPeople"></i>
                <input type="text" tabindex="1" name="loginname" v-model="userName" class="regi_login_input" placeholder="User Name"/>
              </li>
              <li class="regi_form_input noMargin">
                <i class="icon IconPwd"></i>
                <input type="password" tabindex="2" name="password" v-model="userPwd" class="regi_login_input" placeholder="Password"/>
              </li>
            </ul>
          </div>
          <div class="login-wrap">
            <a href="javascript:;" class="btn-login" @click="login">登录</a>
          </div>
        </div>
      </div>
    </div>
    <div class="md-overlay" v-if="loginModalFlag" @click="loginModalFlag=false"></div>
  </header>
</template>

<script>
  import '../assets/css/login.css';
  export default {
    name: "header",
    data() {
      return{
        userName: '',
        userPwd: '',
        errorTip: false,
        loginModalFlag: false
      }
    },
    computed: {
      nickName() {
        return this.$store.state.nickName
      },
      cartCount() {
        return this.$store.state.cartCount
      }
    },
    mounted() {
      this.checkLogin()
    },
    methods: {
      checkLogin(){
        this.axios.get("users/checkLogin").then(res=> {
          var res = res.data;
          if (res.status == "0") {
            this.$store.commit('updateUserInfo',res.result)
            this.getCartCount()
          }else{
            if(this.$route.path!="/goods"){
              this.$router.push("/goods");
            }
          }
        });
      },
      login() {
        if (!this.userName || !this.userPwd) {
          this.errorTip = true
          return
        }
        this.axios.post('/users/login', {
          userName: this.userName,
          userPwd: this.userPwd
        }).then(response => {
          let res = response.data
          if (res.status === '0') {
            this.errorTip = false
            this.loginModalFlag = false
            this.$store.commit('updateUserInfo',res.result.userName)
            this.getCartCount()
          } else {
            this.errorTip = true
          }
        });
      },
      logOut() {
        this.axios.post('/users/logout').then((response) => {
          let res = response.data
          if (res.status === '0') {
            this.$store.commit('updateUserInfo','')
            if(this.$route.path!="/goods"){
              this.$router.push("/goods");
            }
          }
        })
      },
      getCartCount() {
        this.axios.get('/users/cartCount').then(response => {
          let res = response.data
          if(res.status === '0') {
            this.$store.commit('initCartCount', res.result.cartCount)
          }
        })
      }
    }
  }
</script>

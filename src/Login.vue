<script setup>
    import { ref } from 'vue'
    import { useRouter } from 'vue-router'
    import { db } from './data/firebase'
    import { getAuth,signInWithEmailAndPassword } from 'firebase/auth'
    import { doc, getDoc } from 'firebase/firestore'
    import Toast from './components/Toast.vue'

    const email = ref("")
    const password = ref("")
    const message = ref("")
    const router = useRouter()


    const login = async() => {
        try{
            const auth = getAuth()
            const userCredential = await signInWithEmailAndPassword(
                auth,
                email.value,
                password.value
            )

            const user = userCredential.user;

            const userDocRef = doc(db, "users", user.uid)
            const userDocSnap = await getDoc(userDocRef)
            if(userDocSnap.exists()){
                const userData = userDocSnap.data();

                localStorage.setItem('userUid', user.uid);
                localStorage.setItem('username', userData.username || "")
                localStorage.setItem('nickname', userData.nickname || "")
                localStorage.setItem('token', await user.getIdToken())

                window.dispatchEvent(new Event("userChanged"))
                message.value = "登入成功"
                setTimeout(() => router.push('/Member'), 1000)
            }
        }catch(err){
            console.error("登入失敗:", err)
            notify("登入失敗，請確認帳號密碼是否正確")
        }
    }

    const toastRef = ref('')
    const notify = (msg) => {
        toastRef.value.showToast(msg)
    }

</script>

<template>
    <div class="head"></div>
    <div class="container">
        <form @submit.prevent="login">
            <h1>會員登入</h1>
            <div class="input-box">
                <input v-model="email" type="text" name="email" placeholder="登入信箱" required>
                <i class="fa-solid fa-user"></i>
            </div>
            <div class="input-box">
                <input v-model="password" type="password" name="password" placeholder="輸入密碼" required>
                <i class="fa-solid fa-lock"></i>
            </div>
            <button class="login-btn" @click="login">登入</button>
            <p>{{ message }}</p>
            <div class="register-link">
                <p>還沒註冊嗎? <router-link to="/Register">註冊</router-link></p>
            </div>
        </form>
    </div>
    <Toast  ref="toastRef"/>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.head {
  width: 100%;
  height: 70px;
}

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 70px);
  padding: 20px;
  background: radial-gradient(circle,rgba(0, 0, 0, 1) 0%, rgba(175, 214, 224, 1) 29%, rgba(172, 231, 250, 1) 65%, rgba(10, 10, 10, 1) 100%) center center no-repeat;
  background-size: cover;
  animation: bganimation 5s infinite alternate linear;
}

@keyframes bganimation {
  0%{
    background-size: 100% ;
  }
  50%{
    background-size: 150%;
  }
  100%{
    background-size: 200%;
  }
}

form {
  background: rgba(0,0,0,0.8);
  width: 100%;
  max-width: 420px;
  color: white;
  border-radius: 20px;
  padding: 30px;
}

.container:has(form:hover) {
  animation-play-state: paused;
}

.container h1 {
  font-size: clamp(20px, 4vw, 28px);
  text-align: center;
  margin-bottom: 20px;
}

.input-box {
  position: relative;
  width: 100%;
  height: 50px;
  margin: 20px 0;
}

.input-box input {
  width: 100%;
  height: 100%;
  background: transparent;
  border: 2px solid #555;
  border-radius: 40px;
  font-size: 16px;
  color: #fff;
  padding: 0 45px 0 20px;
}

.input-box input::placeholder {
  color: #aaa;
  font-size: 14px;
}

.input-box i {
  position: absolute;
  right: 18px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 18px;
  color: #aaa;
}

.login-btn {
  width: 100%;
  height: 45px;
  background: #fff;
  border: none;
  border-radius: 40px;
  font-size: 16px;
  color: #555;
  font-weight: 600;
  margin-top: 10px;
  transition: .2s ease;
}

.login-btn:hover {
  background: #555;
  color: #fff;
}

.register-link {
  font-size: 14px;
  text-align: center;
  margin-top: 20px;
}

.register-link p a {
  color: #fff;
  text-decoration: none;
  font-weight: 600;
}

.register-link p a:hover {
  text-decoration: underline;
}

@media (max-width: 767px) {
  .container {
    padding: 10px;
  }

  form {
    padding: 20px;
  }

  .input-box {
    margin: 15px 0;
    height: 45px;
  }

  .input-box i {
    font-size: 16px;
    right: 12px;
  }

  .login-btn {
    height: 40px;
    font-size: 15px;
  }
}
</style>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { auth, db } from './data/firebase'
import { createUserWithEmailAndPassword } from 'firebase/auth';
import { doc, setDoc } from 'firebase/firestore'
import { FirebaseError } from 'firebase/app';

const nickname = ref("");
const username = ref("");
const password = ref("");
const email = ref("");
const message = ref("");
const router = useRouter();

const register = async() => {
    if(!nickname.value || !username.value || !password.value || !email.value) {
        message.value = "資料不可空白";
        return
    }

    try {
        const userCredential = await createUserWithEmailAndPassword(auth, email.value, password.value)
        const firebaseUid = userCredential.user.uid

        await setDoc(doc(db, 'users', firebaseUid), {
            nickname: nickname.value,
            username: username.value,
            email: email.value,
            createdAt: new Date()
        })
        console.log("firebase寫入成功")

        localStorage.setItem('userId', firebaseUid)
        localStorage.setItem('nickname', nickname.value)
        localStorage.setItem('username', username.value)

        message.value = '註冊成功'
        setTimeout(() => router.push('/Login'), 1000)

    } catch(err) {
        console.error("firebase 註冊錯誤->", err)
        if(err instanceof FirebaseError){
            switch(err.code){
                case "auth/email-already-in-use":
                    message.value = "此信箱已被註冊"
                    break
                case "auth/invalid-email":
                    message.value = "信箱格式錯誤"
                    break
                case "auth/weak-password":
                    message.value = "密碼至少需要六個字元"
                    break
                default:
                    message.value = "註冊失敗:" + err.message
            }
        } else {
            message.value = "系統錯誤"
        }
    }
}
</script>

<template>
    <div class="head"></div>
    <div class="container">
        <form @submit.prevent="register">
            <h1>會員註冊</h1>
            <div class="input-box">
                <input v-model="nickname" type="text" placeholder="輸入暱稱" required>
                <i class="fa-solid fa-person"></i>
            </div>
            <div class="input-box">
                <input v-model="username" type="text" name="username" placeholder="輸入帳號" required>
                <i class="fa-solid fa-user"></i>
            </div>
            <div class="input-box">
                <input v-model="email" type="email" name="email" placeholder="輸入信箱" required>
                <i class="fa-solid fa-envelope"></i>
            </div>
            <div class="input-box">
                <input v-model="password" type="password" name="password" placeholder="輸入密碼" required>
                <i class="fa-solid fa-lock"></i>
            </div>
            <button type="submit" class="register-btn">註冊</button>
            <p>{{ message }}</p>
            <div class="register-link">
                <p>已經有帳號了? <router-link to="/Login">去登入</router-link></p>
            </div>
        </form>
    </div>
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
  background: radial-gradient(circle,rgba(0, 0, 0, 1) 0%, rgba(224, 187, 175, 1) 29%, rgba(235, 156, 108, 1) 65%, rgba(10, 10, 10, 1) 100%) center center no-repeat;
  background-size: cover;
  animation: bganimation 2s infinite alternate linear;
}

@keyframes bganimation {
  0%{
    background-size: 100% ;
  }
  50%{
    background-size: 140%;
  }
  100%{
    background-size: 180%;
  }
}

form {
  background: #000;
  width: 100%;
  max-width: 420px;
  color: white;
  border: 2px solid #333;
  border-radius: 20px;
  padding: 30px;
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

.login-btn,
.register-btn {
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

.login-btn:hover,
.register-btn:hover {
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

  .login-btn,
  .register-btn {
    height: 40px;
    font-size: 15px;
  }
}
</style>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()
const username = ref("")
const userId = ref("")
const isMenuOpen = ref(false) // 預設關閉

const syncUser = () => {
  username.value = localStorage.getItem("username");
  userId.value = localStorage.getItem("userId");
}

onMounted(() => {
  syncUser()
  window.addEventListener("userChanged", syncUser)
})

onUnmounted(() => {
  syncUser()
  window.removeEventListener("userChanged", syncUser)
})

const logout = () => {
  localStorage.removeItem("username")
  localStorage.removeItem("nickname")
  localStorage.removeItem("userId")
  localStorage.removeItem("token")

  window.dispatchEvent(new Event("userChanged"))

  router.push("/Login")
}
</script>

<template>
  <header>
    <div class="header-left">
      <img src="../assets/images/myLogo.png" alt="#" class="logo">
    </div>

    <!-- 桌機版選單（永遠顯示） -->
    <div class="header-right desktop">
      <ul>
        <span></span>
        <li><router-link to="/" class="link">首頁</router-link></li>
        <span></span>
        <li><router-link to="/attraction" class="link">景點介紹</router-link></li>
        <span></span>
        <li><router-link to="/schedule" class="link">行程規劃</router-link></li>
        <span></span>
        <li v-if="username && route.name ==='Member'">
          <router-link to="/" class="link" @click="logout">登出({{username}})</router-link>
        </li>
        <li v-else-if="!username">
          <router-link to="/login" class="link">會員功能</router-link>
        </li>
        <li v-else>
          <router-link to="/member" class="link">會員專區</router-link>
        </li>
      </ul>
    </div>

    <!-- 手機版選單（漢堡控制顯示） -->
    <div class="header-right mobile" v-if="isMenuOpen">
      <ul>
        <li><router-link to="/" class="link" @click="isMenuOpen = false">首頁</router-link></li>
        <li><router-link to="/attraction" class="link" @click="isMenuOpen = false">景點介紹</router-link></li>
        <li><router-link to="/schedule" class="link" @click="isMenuOpen = false">行程規劃</router-link></li>
        <li v-if="username && route.name ==='Member'" @click="isMenuOpen = false">
          <router-link to="/" class="link" @click="logout">登出({{username}})</router-link>
        </li>
        <li v-else-if="!username">
          <router-link to="/login" class="link" @click="isMenuOpen = false">會員功能</router-link>
        </li>
        <li v-else>
          <router-link to="/member" class="link" @click="isMenuOpen = false">會員專區</router-link>
        </li>
      </ul>
    </div>

    <!-- 漢堡選單 -->
    <div class="hamburger" @click="isMenuOpen = !isMenuOpen">
      <span></span>
      <span></span>
      <span></span>
    </div>
  </header>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

header {
  width: 100%;
  height: 70px;
  position: fixed;
  z-index: 1000;
  background: #000;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-left {
  height: 100%;
}

.header-left img {
  height: 70px;
}

.header-right ul {
  list-style: none;
  display: flex;
  align-items: center;
  gap: 1rem;
  margin: 0 15px;
}

.link {
  color: #fff;
  text-decoration: none;
  font-weight: 900;
}

.header-right ul li:hover {
  transform: scale(1.2);
  transition: transform 0.5s ease;
}

.hamburger {
  display: none;
}

.hamburger span {
  width: 30px;
  height: 4px;
  background: #fff;
  margin: 5px;
  display: block;
}

/* 桌機版 */
.desktop {
  display: flex;
}

/* 手機版 */
.mobile {
  position: absolute;
  top: 70px;
  right: 0;
  width: 200px;
  background: #000;
  flex-direction: column;
  padding: 10px;
}

.mobile ul {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.mobile ul li:hover {
    background: #555;
}

@media (max-width: 768px) {
  .desktop {
    display: none;
  }

  .hamburger {
    display: block;
    cursor: pointer;
    margin-right: 15px;
  }
}
</style>
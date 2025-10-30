<script setup>
    import { ref, onMounted, computed } from 'vue'
    import { db } from './data/firebase'
    import { getAuth } from 'firebase/auth'
    import { collection, addDoc, Timestamp, onSnapshot, query, orderBy } from 'firebase/firestore'
    import Toast from './components/Toast.vue'
    const savedItinerary = ref([])
    const message = ref("")
    const messages = ref([])
    const nickname = ref("")
    const username = ref("")

    onMounted(() => {
        nickname.value = localStorage.getItem('nickname') || ""
        username.value = localStorage.getItem('username') || ""

        if(!nickname.value) return

        const allTrips = JSON.parse(localStorage.getItem("savedTrips")) || {}
        savedItinerary.value = allTrips[username.value] || []

        const q = query(collection(db, "messages"), orderBy("createdAt", "desc"))
        onSnapshot(q, (snapshot) => {
            messages.value = snapshot.docs.map(doc => ({id: doc.id, ...doc.data()}))
        })
    })

    const cleanItinerary = (index) => { 
        if(!username.value)return

        if(!confirm("確定要刪除該行程嗎?")){
            return
        }
        
        savedItinerary.value.splice(index,1)
        const allTrips = JSON.parse(localStorage.getItem('savedTrips')) || {}
        allTrips[username.value] = savedItinerary.value
        localStorage.setItem('savedTrips', JSON.stringify(allTrips))
    }


    const sendMessage = async () => {
        if (!message.value.trim()) {
            notify("請輸入留言");
            return;
        }

        const auth = getAuth()
        const user = auth.currentUser
        if(!user){
            notify("請先登入才能留言")
            return
        }

        try {
            await addDoc(collection(db, 'messages'),{
                uid: user.uid,
                nickname: nickname.value || user.displayName || "訪客",
                username: username.value || user.email,
                message: message.value,
                createdAt: Timestamp.now()
            })
            notify('留言成功')
            message.value = ''
        } catch (err) {
            notify("留言失敗，請稍後再試");
            console.error("firebase錯誤:", err);
        }
    }

    const toastRef = ref('')
    const notify = (msg) => {
        toastRef.value.showToast(msg)
    }

    const duration = computed(() => {
        const len = messages.value ? messages.value.length : 0
        return Math.min(60, Math.max(12, len * 3))
    })

</script>

<template>
    <div class="head"></div>
    <div class="container">
        <div class="bg-blur">
            <div class="marquee" v-if="messages.length">
                <div class="marquee-inner" :style="{ '--duration': duration + 's' }">
                    <div class="marquee-group">
                        <span v-for="msg in messages" :key="msg.id">
                            <strong>{{ msg.nickname || msg.username }}</strong> : {{ msg.message }} &nbsp;&nbsp;|&nbsp;&nbsp;
                        </span>
                    </div>
                    <div class="marquee-group" v-if="messages.length > 1" aria-hidden="true">
                        <span v-for="msg in messages" :key="'dup-' + msg.id">
                            <strong>{{ msg.nickname || msg.username }}</strong> : {{ msg.message }} &nbsp;&nbsp;|&nbsp;&nbsp;
                        </span>
                    </div>
                </div>
            </div>
            <h1 >歡迎回來，{{ nickname || '訪客' }}</h1>
            <div class="member">
                
                <div class="member-left">
                    <h2>我的行程安排</h2>
                    <div class="trip-container" v-if="savedItinerary.length > 0">
                        <div class="savedItinerary" v-for="(trip, index) in savedItinerary" :key="index">
                            <div class="trip-title">
                                <h3>{{ trip.tripName }}</h3>
                                <button class="delBtn" @click="cleanItinerary(index)"><i class="fa-solid fa-xmark"></i></button>
                                <div class="underline"></div>
                            </div>
                            <ul class="triplist">
                                <li v-for="spot in trip.spots" :key="spot.id">
                                    {{ spot.name }}
                                </li>
                            </ul>
                        </div>
                    </div>
                    <p v-else>尚未儲存任何行程</p>
                </div>

                <div class="member-right">
                    <h2>留言板</h2>
                    <textarea v-model="message"></textarea>
                    <button class="saveBtn" @click="sendMessage">發送</button>
                </div>
            </div>
        </div>
    </div>
    <Toast ref="toastRef" />
</template>

<style scoped>
.head {
    width: 100%;
    height: 70px;
}

.container {
    text-align: center;
    background: url('./assets/images/beach01.jpg');
    background-size: cover;
    background-position: 50%;
    height: 100vh;
}

.bg-blur {
    height: 100vh;
    background: rgba(0,0,0,0.1);
    backdrop-filter: blur(10px);
}

.conatiner .marquee {
    width: 100%;
    background: #111;
    overflow: hidden;
    box-sizing: border-box;
    padding: 8px 0;
    font-size: 16px;
}

.marquee .marquee-inner {
    display: flex;
    align-items: center;
    white-space: nowrap;
    animation: marquee-scroll var(--duration, 20s) linear infinite;
    will-change: transform;
    overflow: hidden;

}

.marquee-group {
    display: inline-flex;
    align-items: center;
}

.marquee-group span {
    display: inline-block;
    padding: 0 1rem;
    white-space: nowrap;
    border-right: 1px solid rgba(255, 255, 255, 0,06);
    color: #fff;
    opacity: 0.95;
    font-weight: 500;
}

@keyframes marquee-scroll {
    0% { transform: translateX(0); }
    100% { transform: translateX(-50%); }
}

.marquee-inner:hover {
    animation-play-state: paused;
}

.container h1 {
    font-size: clamp(24px, 4vw, 32px);
    padding: 2rem;
}
.member {
    background: rgba(0,0,0,0.5);
    width: 80%;
    height: 60vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    border-radius: 20px;
    margin: auto;
    padding: 10px;
    color: #fff;
}

.member-left {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    height: 50%;
}

.member-right {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    height: 50%;
}

.trip-container {
    display: flex;;
    flex-wrap: wrap;
    border-radius: 10px;
    gap: 1rem;
}

.savedItinerary {
    background: #555;
    padding: 10px 15px;
    border-radius: 10px;
    min-width: 140px;
    text-align: left;
    flex: 0 1 auto;
}

.trip-title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: relative;
    margin-bottom: 8px;
}

.trip-title h3 {
    font-size: clamp(16px, 4vw, 18px);
    font-weight: 600;
    margin: 0;
}

.trip-title .delBtn {
    width: max-content;
    background: transparent;
    border: none;
    color: #fff;
    font-size: 18px;
    font-weight: 900;
    cursor: pointer;
}

.trip-title .underline {
    width: 80%;
    height: 3px;
    position: absolute;
    bottom: 0;
    background: #fff;
    border-radius: 20px;
}

.triplist {
    display: flex;
    gap: 1rem;
    flex-direction: column;
}

.delBtn:hover {
    transform: scale(1.2);
}

.savedItinerary ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.member-left h2 {
    font-size: clamp(20px, 4vw, 28px);
}

.member-left p {
    font-size: clamp(14px, 5vw, 18px);
}

.member-left img {
    border: 3px solid black;
    border-radius: 20px;
}

.member-right h2 {
    font-size: clamp(20px, 4vw, 28px);
}

.member-right textarea {
    width: 60%;
    height: 100%;
    resize: vertical;
    background: #000;
    color: #fff;
    margin: 10px 40px;
    padding: 20px;
    border: none;
    outline: none;
    border-radius: 20px;
}

.saveBtn {
    padding: 5px 10px;
    border: none;
    border-radius: 20px;
    background: rgba(0,0,0,0.8);
    transition: .2s ease;
    color: #fff;
    font-weight: bold;
}

.saveBtn:hover {
    background: #000;
}

@media (max-width: 767px) {
    .marquee {
        font-size: 14px;
        padding: 6px 0;
    }

    .marquee-group span {
        padding: 0 0.6rem;
    }

    .member {
        flex-direction: column;
        align-items: stretch;
    }

    .member-left, .member-right {
        width: 90%;
        margin: 10px auto;
    }

    .savedItinerary {
        flex: 1 1 100%;
        text-align: left;
    }
}
</style>
<script setup>
    import { ref, computed } from 'vue'
    import attractions from './data/attractions.json'
    import { useRouter } from 'vue-router'
    import draggable from 'vuedraggable'
    import Toast from './components/Toast.vue'

    const selectedId = ref("") //下拉式選單的選項
    const itinerary = ref([])  //行程清單(最多三個)
    const showModal = ref(false)
    const tripName = ref("")
    const router = useRouter()

    const images = import.meta.glob('./assets/images/cardsImg/*', { eager: true, as: 'url' })
    const imageMap = Object.fromEntries(
        Object.entries(images).map(([path, url]) => [path.split('/').pop(), url])
    )


    //取得目前選中的景點
    const selectedSpot = computed(() => {
        return attractions.find(item => item.id === Number(selectedId.value))
    })

    const selectedImg = computed(() => {
        if (!selectedSpot.value) return ''
        return imageMap[selectedSpot.value.image] || ''
    })

    //加入行程
    const addSpot = () => {
        if(!selectedSpot.value) return
        if(itinerary.value.length >= 3){
            notify('最多三個')
            return
        }
            
        //避免重複加入
        if(!itinerary.value.find(s => s.id === selectedSpot.value.id)){
            itinerary.value.push(selectedSpot.value)
        }
    }

    //移除行程中的景點
    const removeSpot = (id) => {
        itinerary.value = itinerary.value.filter(item => item.id !== id)
    } 

    //辨別行程數量是否完整
    const canSave = computed(() => {
        return itinerary.value.length == 3
    })

    const openSaveModal = () => {
        if(itinerary.value.length < 3){
            notify("請先安排完整的行程")
            return
        }
        const username = localStorage.getItem("username")
        if(!username) {
            notify("請先登入")
            router.push('/Login')
        }
        showModal.value = true
    }

    //儲存到localStorage
    const saveItinerary = () => {
        if(!tripName.value.trim()){
            notify('輸入行程名稱')
            return
        }

        const username = localStorage.getItem('username')
        //如果localStorage沒有savedTrips，就給一個空物件
        const allTrips = JSON.parse(localStorage.getItem("savedTrips")) || {}

        if(!allTrips[username]){
            allTrips[username] = []
        }

        allTrips[username].push({
            tripName: tripName.value,
            spots: itinerary.value
        })

        localStorage.setItem("savedTrips", JSON.stringify(allTrips))

        //清空 & 關閉
        tripName.value = ""
        itinerary.value = []
        showModal.value = false
        notify('添加成功')
    }

    const toastRef = ref('')

    const notify = (msg) => {
        toastRef.value.showToast(msg)
    }

</script>

<template>
    <div class="head"></div>
    <div class="container">
        <div class="bg-blur" :style="{backgroundImage: selectedImg ? `url(${selectedImg})` : 'none'}"></div>
        <div class="left-panel">
            <h2>自訂遊覽順序</h2>
            <select v-model="selectedId">
                <option value="" disabled>請選擇景點</option>
                <option v-for="spot in attractions" :key="spot.id" :value="spot.id">
                {{ spot.name }}
                </option>
            </select>
            <div class="left-container" v-if="selectedSpot">
                <div class="image-wrapper">
                    <img :src="selectedImg" :alt="selectedSpot.name" style="max-width: 100%;" />
                </div>
                <button @click="addSpot">加入行程</button>
            </div>
        </div>

        <div class="right-panel">
        <h2>我的行程</h2>
        <h3 class="placeholder" v-show="itinerary.length === 0">尚無行程</h3>
        <draggable v-model="itinerary" item-key="id" animation="500">
            <template #item="{ element }">
            <div class="spot-card">
                <img :src="imageMap[element.image]" :alt="element.name" />
                <div class="spot-info">
                <h3>{{ element.name }}</h3>
                <button @click="removeSpot(element.id)"><i class="fa-solid fa-xmark" style="color: #fff;"></i></button>
                </div>
            </div>
            </template>
        </draggable>
        <button class="saveBtn" v-show="canSave" @click="openSaveModal">保存</button>
        </div>
        <div class="modal-overlay" v-if="showModal" @click.self="showModal = false">
            <div class="modal">
                <p>幫行程命名吧</p>
                <input v-model="tripName" placeholder="輸入行程名稱" maxlength="8">
                <button @click="saveItinerary">儲存</button>
            </div>
        </div>
    </div>
    <Toast  ref="toastRef"/>
</template>

<style scoped>
.container {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    gap: 1rem;
    background: linear-gradient(175deg,rgba(240, 213, 125, 1) 6%, rgba(170, 211, 212, 1) 38%, rgba(131, 210, 230, 1) 73%, rgba(71, 150, 21, 1) 100%, rgba(10, 10, 10, 1) 100%) center center no-repeat;
    background-size: cover;
    background-position: 50%;
}

.head {
    width: 100%;
    height: 70px;
}

.bg-blur {
    position: absolute;
    inset: 0;
    background-size: cover;
    background-position: center;
    filter: blur(30px) brightness(0.7);
    transform: scale(1.1);
    transition: background-image 0.8s ease-in-out, filter 0.8s ease-in-out;
}

.left-panel {
    flex: 2;
    background: rgba(0,0,0,0.5);
    padding: 20px;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    color: #fff;
    margin: 20px;
    backdrop-filter: blur(5px);
}

.left-panel select {
    width: 100%;
    font-size: 16px;
    font-weight: 600;
    padding: 10px;
    border: none;
    border-radius: 20px;
    outline: none;
}

select,::picker(select) {
    appearance: base-select;
    background: #000;
    color: #fff;
    outline: none;
    border: none;
    border-radius: 20px;
    margin-top: 5px;
    padding: 1rem;
}

.left-panel .left-container {
    width: 100%;
    height: 100%;
}

.left-panel .image-wrapper {
    width: 100%;
    height: 300px;
    border-radius: 20px;
    margin: 10px 0;
    overflow: hidden;
    background: #fff;
}

.left-panel img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
}
.left-panel button {
    background: rgba(0,0,0,0.8);
    width: 100%;
    padding: 10px;
    font-size: 16px;
    font-weight: 600;
    color: white;
    border: none;
    border-radius: 20px;
    cursor: pointer;
}
.left-panel button:hover {
    background: rgba(0,0,0,1);
}

.right-panel {
    background: rgba(0,0,0,0.5);
    flex: 3;
    border-radius: 20px;
    color: #fff;
    margin: 20px;
    backdrop-filter: blur(5px);
}

.right-panel h2 {
    margin: 0 auto;
    width: max-content;
}

.right-panel .placeholder {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0 auto;
    font-size: 24px;
    color: #999;
}

.spot-card {
    display: flex;
    align-items: center;
    background: rgba(0,0,0,0.8);
    border-radius: 15px;
    padding: 10px;
    margin: 10px auto;
    width: 90%;
    gap: 15px;
    transition: transform 0.2s ease-in-out;
}
.spot-card:hover {
    transform: scale(1.02);
}
.spot-card img {
    width: 100px;
    height: 80px;
    object-fit: cover;
    border-radius: 10px;
}
.spot-info {
    flex: 1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #fff;
}
.spot-info h3 {
    margin: 0;
    font-size: 18px;
}
.spot-info button {
    background: none;
    border: none;
    font-size: 16px;
    cursor: pointer;
    transition: 0.2s;
}
.spot-info button:hover {
    transform: scale(1.2);
}

.saveBtn {
    position: absolute;
    bottom: 0;
    right: 0;
    margin: 20px;
    padding: 5px 20px;
    background: #555;
    border: none;
    border-radius: 20px;
    color: #fff;
    font-size: 18px;
    font-weight: 600;
}

.saveBtn:hover {
    background: #000;
}

.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.2);
    display: flex;
    justify-content: center;
    align-items: center;
    backdrop-filter: blur(5px);
}

.modal {
    background: rgba(0,0,0,0.8);
    padding: 60px 120px 60px 120px;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2rem;
}

.modal p {
    color: #fff;
    font-size: 24px;
}

.modal input {
    font-size: 18px;
    border: none;
    border-radius: 20px;
    padding: 5px 10px 5px 10px;
    outline: none;
}

.modal button {
    font-size: 16px;
    border: none;
    padding: 5px 10px 5px 10px;
    background: #fff;
    border-radius: 20px;
    transition: 0.2s;
}

.modal button:hover {
    transform: scale(1.05);
}

@media (max-width: 1199px) {
        .container{
        display: flex;
        flex-direction: column;
    }
}

@media (max-width: 767px) {
    .container{
        display: flex;
        flex-direction: column;
    }

    .saveBtn {
        position: absolute;
        margin: 0;
    }
}

</style>
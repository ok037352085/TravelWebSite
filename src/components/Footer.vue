<script setup>
  const now = new Date().getFullYear()

</script>

<template>
    <footer>
        <div class="footer-container">
            <div class="footer-top">
                <div class="follow">
                    <div>
                        <p>FOLLOW</p>
                    </div>
                    <div class="icon">
                        <a class="line" href="https://lin.ee/3I8FR3n"><i class="fa-brands fa-line"></i></a>
                        <a class="instagram" href="https://www.instagram.com"><i class="fa-brands fa-square-instagram"></i></a>
                        <a class="github" href="https://github.com/ok037352085"><i class="fa-brands fa-square-github"></i></a>
                    </div>
                </div>
            </div>
            <div class="footer-bottom">
                <p>Copyright © {{ now }}</p>
            </div>
        </div>
    </footer>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100..900;1,100..900&family=Rubik:ital,wght@0,300..900;1,300..900&display=swap');

.footer-container {
  background: black;
  bottom: 0;
  margin: 0;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  color: #999;
  padding: 20px 10px;
  box-sizing: border-box;
}

.footer-container .footer-top {
  max-width: 800px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
}

.follow {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.follow p {
  font-family: "Roboto", sans-serif;
  font-size: clamp(14px, 2vw, 20px);
}

.follow .icon {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 10px;
}

.follow .icon a {
  font-size: 2rem;
  cursor: pointer;
  color: #999;
}

.footer-middle {
  max-width: 800px;
  width: 100%;
  margin-bottom: 20px;
}

.footer-middle ul {
  width: 100%;
  list-style: none;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 12px;
  font-size: clamp(12px, 2vw, 18px);
  font-family: "Roboto", sans-serif;
  padding: 0;
}

.footer-middle ul li {
  cursor: pointer;
  color: #fff;
  text-decoration: none;
}

.footer-middle ul span {
  background: #999;
  width: 2px;
  height: 18px;
}

.footer-bottom {
  max-width: 800px;
  width: 100%;
  display: flex;
  justify-content: center;
}

.footer-bottom p {
  font-size: clamp(12px, 1.5vw, 16px);
  margin: 0;
}

@media (max-width: 600px) {
  .footer-middle ul {
    flex-direction: column;
    gap: 8px;
  }
  .footer-middle ul span {
    display: none;
  }
}
</style>

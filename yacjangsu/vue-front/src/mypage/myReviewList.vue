<template>
  <div class="con">
    <div class="info">
      <div class="my">
        <div style="margin-left: 30px; margin-right: 200px;">
          <h2 style="color: #fff;">{{ loginUser.user_name }}님의</h2>
          <h2 style="color: #fff;">회원정보입니다.</h2>
        </div>

        <div class="infobox">
          <div class="userinfo">
            <h1 class="count">{{ userOrder.order_cnt }}</h1>
            <h3>주문내역</h3>
          </div>
        </div>

        <div class="infobox">
          <div class="userinfo">
            <h1 class="count">{{ loginUser.user_point }}</h1>
            <h3>적립금</h3>
          </div>
        </div>

        <div class="infobox">
          <div class="userinfo">
            <h1 class="count">{{ userDeli.deli_cnt }}</h1>
            <h3>배송중</h3>
          </div>
        </div>
      </div>
    </div>

    <div class="container">
      <nav>
        <div class="sidebar">
          <div class="icon-bar mypage-bar">
            <div>
              <router-link class="m" to="/mypage">
                <button style="background-color: #fff; border-style: none;">
                  <font-awesome-icon icon="fa-solid fa-address-card" size="xl" style="color: #9c0410;" />
                  <p>회원 정보</p>
                </button>
              </router-link>
            </div>
            <div>
              <router-link to="/mypage/myOrderList" class="m">
                <button style="background-color: #fff; border-color: #fff; border-style: none;">
                  <font-awesome-icon icon="fa-solid fa-credit-card" size="xl" style="color: #9c0410;" />
                  <p>주문 내역</p>
                </button>
              </router-link>
            </div>
            <div>
              <router-link to="/mypage/myReviewList" class="m">
                <button style="background-color: #fff; border-color: #fff; border-style: none;">
                  <font-awesome-icon icon="fa-solid fa-pen-to-square" size="xl" style="color: #9c0410;" />
                  <p>나의 리뷰</p>
                </button>
              </router-link>
            </div>

            <div>
              <router-link to="/mypage/myQnaList" class="m">
                <button style="background-color: #fff; border-color: #fff; border-style: none;">
                  <font-awesome-icon icon="fa-solid fa-circle-question" size="xl" style="color: #9c0410;" />
                  <p>문의 내역</p>
                </button>
              </router-link>
            </div>

            <div>
              <router-link to="/mypage/wishList" class="m">
                <button style="background-color: #fff; border-color: #fff; border-style: none;">
                  <font-awesome-icon :icon="['fas', 'heart']" size="xl" style="color: #9c0410;" />
                  <p>찜한 상품</p>
                </button>
              </router-link>
            </div>

          </div>

        </div>
      </nav>
      <!-- <section> -->
      <div class="myinfo">
        <p style="text-align: center; font-size:170%; margin: 0 100px 60px 0;">리뷰 목록</p>
        <div class="empty" v-if="reviewList.length == 0">
          <p>아직 작성한 리뷰가 없습니다. <br>
            주문 내역에서 리뷰를 작성해주세요. </p>
        </div>
        <div v-else class="cardList">
          <div class="card" v-for="(review, index) in pageReviewList" :key="index">
            <div class="card-body">
              <div class="card-header">
                <img v-if="!review.REVIEW_IMG" src="../assets/imgEmptyInput.png" alt="...">
                <img v-else :src="require(`../../../node-back/uploads/uploadReview/${review.REVIEW_IMG}`)" alt="..."
                  :width="300">
              </div>
              <div class="card-body-header">
                <a @click="goToDetail(review.GOODS_NO)">
                  <h1>{{ review.GOODS_NAME }}</h1>
                </a>
                <div class="star-rating">
                  <div class="star" v-for="score in 5" :key="score">
                    <span v-if="score < review.REVIEW_SCORE + 1"><i class="fa fa-star"></i></span>
                    <span v-if="score >= review.REVIEW_SCORE + 1"><i class="fa fa-star-o"></i></span>
                  </div>
                  <span style="margin-left: 10px;">({{ review.REVIEW_SCORE }})</span>
                </div>
              </div>
              <div class="card-body-description">
                <p>{{ review.REVIEW_CON }}</p>
              </div>
              <div class="card-body-footer">
                <p>{{ formatDateTime(review.REVIEW_DATE) }}</p>

                <a href="#" @click.prevent="reviewUpdate(review.REVIEW_NO)" style="text-decoration: none; color: red;">
                  <h1>수정</h1>
                </a>

                <a href="#" @click.prevent="reviewDelete(review.REVIEW_NO)" style="text-decoration: none; color: gray;">
                  <h1 style="margin-left: 10px;">삭제</h1>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- </section> -->
      <nav class="page">
        <ul>
          <li v-for="i in pageCnt" :key="i" class="page-item">
            <a @click.prevent="setPage(i)" style="text-decoration: none;">
              {{ i }}
            </a>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>
<script>
import axios from 'axios';

export default {
  data() {
    return {
      // num: 0,
      loginUser: {},
      userOrder: {},
      userDeli: {},
      reviewList: [],
      pageReviewList: [],
      pageNum: 0,
      pageCnt: 0,
      onePageCnt: 6,
    };
  },
  computed: {
    user() {
      return this.$store.state.user;
    }
  },
  mounted() {
    if (this.user.user_no == '') {
      alert('로그인해주세요');
      this.$router.push({ path: '/login' });
    }
  },
  created() {
    this.getUser();
    this.getUserOrder();
    this.getUserDeli();
    this.getReviewList();
  },
  methods: {
    async getUser() {
      try {
        const response = await axios.get(`http://localhost:3000/mypage/mypage/${this.user.user_no}`);
        this.loginUser = response.data[0];
      } catch (error) {
        console.error(error);
      }
    },
    async getUserOrder() {
      try {
        const response = await axios.get(`http://localhost:3000/mypage/orderbox/${this.user.user_no}`);
        this.userOrder= response.data[0];
      } catch (error) {
        console.error(error);
      }
    },
    async getUserDeli() {
      try {
        const response = await axios.get(`http://localhost:3000/mypage/delibox/${this.user.user_no}`);
        this.userDeli= response.data[0];
      } catch (error) {
        console.error(error);
      }
    },
    setPage(page) {
      this.pageReviewList = []
      this.pageNum = page - 1;
      this.sliceList()
    },
    sliceList() {
      const start = 0 + this.pageNum * this.onePageCnt
      this.pageReviewList = this.reviewList.slice(start, start + this.onePageCnt);
    },
    async getReviewList() {
      try {
        const response = await axios.get(`http://localhost:3000/mypage/getMyReview2/${this.user.user_no}`);
        this.reviewList = response.data;
        this.pageCnt = Math.ceil(this.reviewList.length / this.onePageCnt);
        this.setPage(1); // 첫 페이지로 설정
        console.log(this.reviewList);
      } catch (error) {
        console.error(error);
      }
    },
    formatDateTime(dateTime) {
      const date = new Date(dateTime);
      const options = {
        year: "numeric",
        month: "long",
        day: "numeric",
      };
      const formattedDateTime = date.toLocaleDateString("ko-KR", options);
      return formattedDateTime;
    },
    goToDetail(goodsno) {
      window.location.href = `http://localhost:8080/goods/goodsDetail/${goodsno}`;
    },
    reviewUpdate(reviewno) {
      window.location.href = `http://localhost:8080/reviewUpdate/${reviewno}`;
    },
    reviewDelete(reviewno) {
      this.$swal({
        title: `해당 리뷰를 삭제하시겠습니까?`,
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: '삭제',
        cancelButtonText: '취소',
        reverseButtons: true,
      }).then(result => {
        const review = this.reviewList.find(review => review.REVIEW_NO === reviewno);
        if (result.value) {
          axios({
            url: `http://localhost:3000/mypage/reviewDelete/${reviewno}/${review.ORDERS_NO}`,
            data: {
              review_no: reviewno,
            },
            method: 'POST'
          })
            .then(res => {
              if (res.data.message === 'complete') {
                this.$swal('리뷰가 삭제되었습니다.');
                this.getReviewList(); // 삭제 후 목록 다시 불러오기
              }
            })
            .catch(() => {
              this.$swal('오류 발생');
            });
        }
      })

    },
  }
}
</script>
<style scoped>
@charset "UTF-8";

/*폰트*/
@font-face {
  font-family: 'ChosunCentennial';
  src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2206-02@1.0/ChosunCentennial.woff2') format('woff2');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'seolleimcool-SemiBold';
  src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2312-1@1.1/seolleimcool-SemiBold.woff2') format('woff2');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'SejonghospitalBold';
  src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2312-1@1.1/SejonghospitalBold.woff2') format('woff2');
  font-weight: 700;
  font-style: normal;
}

@font-face {
  font-family: 'GmarketSansMedium';
  src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2001@1.1/GmarketSansMedium.woff') format('woff');
  font-weight: normal;
  font-style: normal;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

hr {
  border: 2px solid #9c0410;
  width: 550px;
  margin-top: 10px;
}

h2 {
  margin: 20px 0;
}

ul,
li {
  list-style: none;
}

a {
  text-decoration: none;
}

:root {
  --app-container: #f7f8fa;
  --main-color: #1f1c2e;
  --secondary-color: #4A4A4A;
  --link-color: #1f1c2e;
  --link-color-hover: #c3cff4;
  --link-color-active: #fff;
  --link-color-active-bg: #1f1c2e;
  --projects-section: #fff;
  --message-box-hover: #fafcff;
  --message-box-border: #e9ebf0;
  --more-list-bg: #fff;
  --more-list-bg-hover: #f6fbff;
  --more-list-shadow: rgba(209, 209, 209, 0.4);
  --button-bg: #1f1c24;
  --search-area-bg: #fff;
  --star: #1ff1c2e;
  --message-btn: #fff;
}

body {
  font-family: 'SejonghospitalBold';
  overflow: scroll;
  background-color: var(--app-container);
}

/*스크롤*/
body::-webkit-scrollbar {
  width: 12px;
  height: 8px;
}

body::-webkit-scrollbar-track {
  background-color: rgb(218, 218, 218);
  border-radius: 10px;
}

body::-webkit-scrollbar-thumb {
  border-radius: 5px;
  background-color: #9c0410;
}

body::-webkit-scrollbar-button {
  width: 0;
  height: 0;
}


/* 회원정보 상단 부분 */

.con {
  justify-content: space-between;
  align-items: center;
  flex: 2;
  background-color: var(--projects-section);
  border-radius: 32px;
  /* padding: 50px; */
  margin: 50px;
  /* display: flex; */
  flex-direction: column;
  height: 1400px;
}

.my {
  display: flex;
  width: 100%;
  position: relative;
  justify-content: space-between;
  align-items: center;
  padding-right: 20px;
}


.info {
  font-family: 'GmarketSansMedium';
  background-color: #9c0410;
  width: 100%;
  height: 200px;
  justify-content: space-between;
  align-items: center;
  flex: 2;
  border-radius: 32px 32px 0px 0px;
  padding: 40px;
  display: flex;
  flex-direction: column;
}

.infobox {
  background-color: #fff;
  width: 260px;
  height: 130px;
  padding: 30px;
}

.infobox h1 {
  margin-bottom: 10px;
}

.userinfo {
  text-align: center;
}

.userinfo .count {
  padding-bottom: 4px;
}

/* 네비게이션 바 */

.container {
  margin: 40px 30px;
  font-family: 'SejonghospitalBold';
}

.sidebar {
  width: 100px;
}

nav {
  float: left;
  margin-left: 40px;
  margin-right: 100px;
}

nav p {
  font-family: 'seolleimcool-SemiBold';
}

.mypage-bar {
  overflow-y: scroll;
  -ms-overflow-style: none;
  scrollbar-width: none;
}

.mypage-bar::-webkit-scrollbar {
  display: none;
}

.mypage-bar {

  width: 100px;
  top: 150px;
  margin-top: 20px;
  left: 0;
  overflow-x: hidden;
}

.mypage-bar .m,
.icon-bar p {
  color: #9c0410;
}

.mypage-bar p {
  margin: auto;
  font-size: 20px;
  padding-top: 4px;
  margin-bottom: 80px;
}

.container {
  margin: 40px 30px;
}

h2 {
  margin: 20px 0;
}

/* 리뷰카드 */

.empty {
  border-radius: 10px;
  height: 200px;
  text-align: center;
  margin: 0 100px 60px 200px;
}

.empty p {
  color: black;
  font-size: 16px;
  text-align: center;
  padding-top: 80px;
}

.cardList {
  margin: -40px;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  margin-bottom: 50px;
  margin-left: 330px;
}

.card {
  height: 500px;
  width: 300px;
  border-radius: 15px;
  display: inline-block;
  position: relative;
  border: 2px solid #C52922;
  overflow: hidden;
  scale: 0.8;

}

.card:hover {
  transform: scale(1.04);
  transition: all 0.06s linear;
}

.card-header {
  transition: 0.5s;
  width: 100%;
  height: 300px;
  border-radius: 15px 15px 0 0;
  background-size: 100% 280px;
  background-repeat: no-repeat;
}

.card-header img {
  margin: 10px 10px 10px 10px;
  width: 275px;
  border-radius: 10px;
}

.card-body-header {
  line-height: 25px;
  margin: 10px 20px 0px 20px;

}

.card-body-header h1 {
  font-size: 19px;
}

.star-rating {
  display: flex;
  padding-top: 9px;
  font-size: 20px;
  color: #9c0410;
}

.card-body-description {
  margin: 10px 20px;
  font-size: 14px;
  overflow: hidden;
  max-height: 70px;
}

.card-body-description p {
  font-size: 20px;
}

.card-body-footer {
  position: absolute;
  display: flex;
  margin-top: 15px;
  margin-bottom: 6px;
  bottom: 0;
  left: 5px;
  width: 314px;
  font-size: 14px;
  color: #9FA5A8;
  padding: 0 15px;
}

.card-body-footer p {
  margin-top: 3px;
}

.card-body-footer h1 {
  margin-left: 60px;
  text-align: center;
  font-size: 23px;
}

/*페이지*/

.page {
  text-align: center;
  width: 107%;
}

.page-item {
  display: inline-block;
  margin-right: 30px;
}
</style>
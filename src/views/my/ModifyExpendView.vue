<template>
    <div class="container">
      <div class="set">
        <div class="classification">항목명</div>
        <input class="name" type="text" placeholder="항목명을 입력해주세요." v-model="title" />
      </div>
      <div class="line"></div>
  
      <div class="set">
        <div class="classification">정기지출일</div>
        <input class="day" type="number" min="1" max="31" placeholder="정기 지출일을 숫자만 입력해주세요." v-model="selectedDay" />일
      </div>
      <div class="line"></div>
  
      <div class="set">
        <div class="classification">금액</div>
        <input class="balance" type="number" min="0" placeholder="금액을 숫자만 입력해주세요." v-model="fee" />원
      </div>
      <div class="line"></div>
  
      <div class="set">
        <div class="classification">연결된 자산</div>
        <select name="account" class="connected" v-model="selectedAsset">
          <option v-for="asset in assets" :key="asset.id" :value="asset.name">{{ asset.name }}</option>
          <!-- <option value="삼성카드">삼성카드</option>
          <option value="현대 M 체크카드">현대 M 체크카드</option>
          <option value="우리카드">우리카드</option>
          <option value="하나카드">하나카드</option> -->
        </select>
      </div>
  
      <div class="set">
        <div class="classification">지출 카테고리</div>
        <div class="category-container">
          <!-- 메인 카테고리 선택 -->
          <select name="main-category" class="category" v-model="selectedCategory">
            <option value="------">------</option>
            <option value="여가">여가</option>
                    <option value="통신비">통신비</option>
                    <option value="교통비">교통비</option>
                    <option value="쇼핑">쇼핑</option>
                    <option value="식비">식비</option>
                    <option value="생활">생활</option>
                    <option value="공부">공부</option>
                    <option value="의료">의료</option>
                    <option value="여행">여행</option>
          </select>
  
          <!-- 서브 카테고리 선택 (메인 카테고리에 따라 다르게 표시) -->
          <select name="sub-category" class="category" v-model="selectedSubCategory">
            <option v-for="subCategory in filteredSubCategories" :key="subCategory" :value="subCategory">
              {{ subCategory }}
            </option>
          </select>
        </div>
      </div>
    </div>
  
    <RouterLink to="/my/expend/">
      <button @click="updateExpense" class="save">수정</button>
    </RouterLink>
  </template>
  
  <script setup>
  import { ref, onMounted, computed } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  
  const router = useRouter();
  const route = useRoute();
  const expenseId = route.params.id; // URL에서 ID를 추출

  const assets = ref([]);
    onMounted(async () => {
        const response = await fetch('http://localhost:8080/asset');
        assets.value = await response.json();
    })
  
  // State variables
  const title = ref('');
  const selectedDay = ref('');
  const fee = ref('');
  const selectedAsset = ref('');
  const selectedCategory = ref('');
  const selectedSubCategory = ref('');
  
  // 메인 카테고리와 연결된 서브 카테고리 맵핑
  const subCategories = {
    '여가': ['영화', '공연', '게임', '노래방'],
    '통신비': ['인터넷', '모바일 요금'],
    '교통비': ['버스', '지하철', '택시'],
    '쇼핑': ['의류', '선물', '화장품', '생활용품', '전자제품'],
    '식비': ['일식', '양식', '커피', '패스트푸드'],
    '생활': ['뷰티', '스포츠', '반려동물', '관리비', '인터넷', '웰세'],
    '공부': ['책', '온라인 강좌'],
    '의료': ['약국', '병원'],
    '여행': ['티켓', '교통', '숙박', '식비']
  };
  
  // 선택된 메인 카테고리에 맞는 서브 카테고리를 반환
  const filteredSubCategories = computed(() => {
    return subCategories[selectedCategory.value] || ["------"];
  });
  
  onMounted(async () => {
    try {
      const response = await fetch(`http://localhost:8080/regular-expense/${expenseId}`);
      const data = await response.json();
      title.value = data.title;
      selectedDay.value = data.day;
      fee.value = data.fee;
      selectedAsset.value = data.assetName;
      selectedCategory.value = data.categoryName;
      selectedSubCategory.value = data.subCategory || '------';
    } catch (error) {
      console.error('데이터 불러오기 중 오류 발생:', error);
      alert('데이터를 불러오는 데 실패했습니다.');
    }
  });
  
  const updateExpense = async () => {
    const userNickname = localStorage.getItem('nickname');
    const updatedExpense = {
      nickname: userNickname,
      title: title.value,
      categoryName: selectedCategory.value,
      day: parseInt(selectedDay.value),
      assetName: selectedAsset.value,
      fee: parseInt(fee.value),
      subCategory: selectedSubCategory.value,
    };
  
    try {
      const response = await fetch(`http://localhost:8080/regular-expense/${expenseId}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updatedExpense),
      });
  
      if (response.ok) {
        alert('정기지출이 성공적으로 수정되었습니다.');
        router.push('/my/expend');
      } else {
        alert('정기지출 수정에 실패했습니다.');
      }
    } catch (error) {
      console.error('정기지출 수정 중 오류 발생:', error);
      alert('정기지출 수정 중 오류가 발생했습니다.');
    }
  };
  </script>
<style scoped>
    .container {
    display: flex;
    flex-direction: column;
    margin-top: 80px;
}

.set {
    display: flex;
    flex-direction: row;
    margin-top: 66px;
    align-items: center;
}

.classification {
    font-family: 'Noto Sans KR';
    font-style: normal;
    font-weight: 500;
    font-size: 22px;
    line-height: 26px;
    color: #101424;
    width: 200px; 
}

.name, .balance, .day, .connected, .category {
    width: 560px;
    height: 29px;
    margin-left: 40px; 
    font-family: 'Noto Sans KR';
    font-style: normal;
    font-weight: 300;
    font-size: 18px;
    line-height: 22px;
    color: #101424;
    border: none;
}

.line {
    width: 540px;
    margin-left: 240px;
    margin-top: 2px;
    background: #B1B1D2;
    border: 1px solid #B1B1D2;
}

.connected, .category {
    height: 40px;
    border: 1px solid #B1B1D2;
    border-radius: 5px;
}

.save {
    width: 84px;
    height: 48px;
    margin-left: 740px;
    margin-top: 70px;
    background: #101424;
    border-radius: 13px;
    font-family: 'Noto Sans KR';
    font-style: normal;
    font-weight: 350;
    font-size: 24px;
    line-height: 26px;
    color: #FFFFFF;
    cursor: pointer;
}

.category-container {
    display: flex;
    gap: 10px; 
}

.category {
    width: 260px; 
    height: 40px;
    border: 1px solid #B1B1D2;
    border-radius: 5px;
    padding: 5px 10px;
    background-color: #ffffff;
    font-family: 'Noto Sans KR';
    font-size: 18px;
    color: #101424;
}
</style>
# 코드 설명

### 1-(1).final_code_데이터수집.ipynb
- 카카오 API(키워드 검색)을 통해 기본 데이터 획득 : 카테고리, 장소명, 위치, URL 등
- Selenium을 통해 데이터 추가 획득 : 평점, 이미지 정보
- 검색 기준
  - `keywords`(총 10개): ['맛집', '분위기 좋은', '테마파크' ,'오션뷰', '감성', '가족여행', '체험', '휴식', '레포츠', '가볼만한 곳']
  - `categorys`(총 4개) : {'CT1' : '문화시설', 'AT4' : '관광명소', 'FD6' : '음식점', 'CE7' : '카페'}
    - '문화시설'의 경우 개수가 현저히 적게 수집되어 '관광명소'로 변경
- `서귀포시에 대한 데이터를 누락하여 추가적으로 수집`
- 중복된 데이터 처리 : 'id' 컬럼 기준으로 중복 제거
---
### 1-(2).final_code_숙박데이터 수집.ipynb
- 카카오 API(키워드 검색)을 통해 기본 데이터 획득 : 카테고리, 장소명, 위치, URL 등
- Selenium을 통해 데이터 추가 획득 : 평점, 등급, 이미지 정보
- 검색 기준
  - `keywords`(총 6개): ['호텔', '리조트', '콘도', '게스트하우스', '민박', '펜션']
  - `categorys`(총 1개) : {'AD5' : '숙박'}
- `서귀포시에 대한 데이터를 누락하여 추가적으로 수집`
- 중복된 데이터 처리 : 'id' 컬럼 기준으로 중복 제거
- 적게 수집된 `리조트`와 `코도`를 합쳐 `리조트/콘도`로 변경
---
### 1-(3).final_code_데이터 처리.ipynb
- 이미지를 확인하여 해당 장소와 상관없는 이미지일 경우 삭제
  - 관광지(관광명소, 음식점, 카페) 데이터 : 3008개
  - 숙박 데이터 : 3004개
- 이미지 사이즈 조절 : 224 X 224
  - ImageNet기반 사전 학습 모델 사용을 위해 미리 사이즈 조절
- 자연어 처리 : 코사인 유사도
  - content 컬럼 : keyword와 category_group_name을 합친 자연어 말뭉치
  - CountVectorizer 활용 -> Count Vector 생성
  - Cosine Similarity를 활용해 유사도 계산
  - 추후 활용을 위해 `.npy` 형태로 저장
---
### 2.final_code_img_feature extraction.ipynb
- **VGG16을 사용한 Feature Extraction**
  - 음식점, 카페의 경우 특징을 추출하기 어려운 부분이 있어 **`관광명소`**만 사용
  - 특성 벡터로 유클리드 거리/코사인 유사도 비교
- 유클리드 거리와 코사인 유사도의 크게 보이지 않아 `유클리드 거리`만 사용
- 추후 활용을 위해 `.npy` 형태로 저장
---
### 3.final_code_음식 라벨링_from CLIP.ipynb
- 이미지로 식당의 라벨링을 진행하기 위해 CLIP 사용
- 이미지를 보며 최대한 여러 음식을 묶을 수 있도록 음식 카테고리 선정
- CLIP을 최대로 활용하기 위해 카테고리는 영어로 작성
- 음식 카테고리(32 개) : ["chicken", "Grilled pork","chiness food","sea food","noodle","Grilled fish", "Grilled Cutlassfish","beef","Hanjeongsik", "sashimi","bolied pork","hamburger","shrimp sashimi","shrimp","melon prosciutto","pork cutlet", "bread","pizza","Waffle","Tteokbokki","pasta","ramen","sushi","corn dog","American breakfast","crab","curry","bread","soup","tuna", "doughnut","koreanstyle sushi(gimbab)"]
---
### 4.final_code_카카오 네비게이션API_Test.ipynb
- 카카오 네비게이션 API를 활용해 경로 획득
- 경로의 좌표를 이용해 folium으로 출력
---
### 5.final_code_이미지유사도.ipynb
- VGG16을 사용해 특징 벡터 추출
- 유클리드 거리와 코사인 유사도의 크게 보이지 않아 `유클리드 거리`만 사용
- 추후 활용을 위해 `.npy` 형태로 저장
---
### 6.final_code_주변장소 거리 및 개수.ipynb
- 특정 좌표간 직선 거리 함수 구현
  - 우리나라 기준 1도 = 88.8km, 1분 = 1.48km, 1초 = 25m
  - 위도 차이 : ((a.lat - b.lat) * 88.8) km
  - 경도 차이 : ((a.lng - b.lng) * 88.8) km
  - 거리 차이 : (위도 차이 2 + 경도 차이 2) ** (1/2) km
- 직선 거리 함수를 이용해 5km 이내의 주변 관광지 개수 컬럼 추가
---

# 🛫제주도 관광지 추천 모델 개발
## Introduction
- **제주도 관광지 추천 모델**은 관광지를 쉽게 선정하기위한 모델입니다.
- 이미지, 자연어(카테고리, 키워드 등)을 이용해 유사도를 계산하고 관광지를 추천하는 시스템을 구현하였습니다.
- 구현 결과(왼쪽 : 네비게이션 API 최적 경로 | 오른쪽 : 직선 거리 최적 경로)
![image](https://user-images.githubusercontent.com/90487843/151348802-5ed45d09-c912-4ff7-bf64-b4204ad72f94.png)

## Contents
- [1. 프로젝트 소개](#1-프로젝트-소개) 
- [2. 데이터 수집 및 전처리](#2-데이터-수집-및-전처리)
  - [관광지 데이터](#관광지-데이터)
  - [숙박 데이터](#숙박-데이터)
- [3. 모델링](#3-모델링)
  - [이미지 특징 추출 및 유사도 측정](#이미지-특징-추출-및-유사도-측정)
  - [음식 라벨링](#음식-라벨링)
  - [테마 자연어 유사도](#테마-자연어-유사도)
  - [경로 추천](#경로-추천)
- [4. 최종 시스템](#4-최종-시스템)
- [5. 한계 및 보완점](#5-한계-및-보완점) 
- [6. 참고 자료](#6-참고-자료) 
- [7. 구성 인원](#7-구성-인원) 
## 1. 프로젝트 소개
### 배경
- 가고 싶은 장소는 많지만 일일이 장소를 확인하며 선정하기 어려움
- 장소를 선정하더라도 순서에 따라 시간과 거리가 달라져 최적의 경로 선정이 어려움
- 장소와 경로에 대한 추천으로 해당 과정의 어려움을 줄일 수 있을 것

### 프로젝트 개요
- 구성인원 : 김준형, 김남규
- 수행기간 : 1달 (2022년 1월)
- 목표 : 키워드와 테마 선정으로 관광지와 경로 추천
- 데이터 : 카카오 키워드 검색 API, selenium 활용
  - 대표 컬럼 : [**id, place_name, keyword, category_group_name, x, y, base_url, rating, (image)**]
  - 관광지 데이터(3008개) 
  - 숙박 데이터(3004개)

### 개발 환경
- 언어 : Python
- 라이브러리 : **Jupyter**, **Pandas**, **Numpy**, **Selenium**, **Scikit-Learn**, **Tensorflow**
- 알고리즘 : **VGG16**, **CLIP**

## 2. 데이터 수집 및 전처리
### 관광지 데이터
#### ▪데이터 수집
- 검색 키워드(9개) : '맛집', '분위기 좋은', '테마파크', '오션뷰', '감성', '가족여행', '체험', '휴식', '레포츠', '가볼만한 곳'
- 카테고리(4개) : CT1(문화시설), AT4(관광명소), FD6(음식점), CE7(카페)
- (제주시, 서귀포시) 법정동, 리 별로 검색

![image](https://user-images.githubusercontent.com/90487843/151355026-f045e64f-8efe-4c3a-8f4a-eb6a7a754c9e.png)
![image](https://user-images.githubusercontent.com/90487843/151355215-4f66c54c-3194-455a-98e7-d3f5dd570a1f.png)
#### ▪데이터 전처리
- id 중복값 처리  
![image](https://user-images.githubusercontent.com/90487843/151356463-7ea194cc-dd72-4cb6-91fd-e167201b5f92.png)
- 문화 시설의 수가 다른 카테고리에 비해 현저히 적어 문화시설은 관광명소로 변경
- 이미지가 없는 데이터 제거
- 이미지의 사이즈 (224 X 224)로 조절 : ImageNet 기반의 사전 학습 모델 사용  
=> **총 데이터 : 3008개**
### 숙박 데이터
#### ▪데이터 수집
- 검색 키워드(6개) : '호텔', '리조트',' 콘도', '게스트하우스', '민박', '펜션'
- 카테고리(1개) : AD5(숙박)
- (제주시, 서귀포시) 법정동, 리 별로 검색

![image](https://user-images.githubusercontent.com/90487843/151356092-c55aa704-6f07-4f6d-a230-92a86e188d36.png)
![image](https://user-images.githubusercontent.com/90487843/151356185-e56a836e-98cf-4d0f-818a-e5dc0c055f7e.png)
#### ▪데이터 전처리
- id 중복값 처리  
![image](https://user-images.githubusercontent.com/90487843/151356647-b73b8931-3462-4a98-a6f0-7717c604676e.png)
- 수가 상대적으로 적은 리조트와 코드를 리조트/콘도로 병합
- 이미지가 없는 데이터 제거
=> **총 데이터 : 3004개**

## 3. 모델링
### 이미지 특징 추출 및 유사도 측정
- VGG16 모델을 활용해 특징 추출(Feature Extraction) : 
  - 첫 번째 FC Layer 추출 : (1, 4096) 형태의 특징 벡터 획득
![image](https://user-images.githubusercontent.com/90487843/151359722-3ba9b420-eaa1-4832-adc5-003c996660c2.png)
- 유클리드 거리
![image](https://user-images.githubusercontent.com/90487843/151360717-bded336e-371c-469b-8511-5e96e7d410c1.png)
- 코사인 유사도
![image](https://user-images.githubusercontent.com/90487843/151360736-4fac178b-861d-4713-b710-bfc17b391934.png)

- **유클리드 거리와 코사인 유사도의 차이가 보이지 않아 최종적으로 `코사인 유사도` 사용**
- 음식점과 카페는 특징 추출로는 표현이 잘되지 않아 **관광 명소**만 진행

### 음식 라벨링
- CLIP : 이미지와 텍스트 쌍을 예측하도록 학습되는 모델
![image](https://user-images.githubusercontent.com/90487843/151361671-e1438075-23f0-4655-999c-f5dfd8eb0029.png)
- 음식 카테고리 선정 (32개)
  - CLIP을 활용하기 위해 영어로 작성
  - ["chicken", "Grilled pork","chiness food","sea food","noodle","Grilled fish", "Grilled Cutlassfish","beef","Hanjeongsik", "sashimi","bolied pork","hamburger","shrimp sashimi","shrimp","melon prosciutto","pork cutlet", "bread","pizza","Waffle","Tteokbokki","pasta","ramen","sushi","corn dog","American breakfast","crab","curry","bread","soup","tuna", "doughnut","koreanstyle sushi(gimbab)"]
- 음식 이미지와 음식 카테고리로 라벨링 진행
  - 가장 높은 예측을 보여주는 라벨링을 음식의 카테고리로 선정
![image](https://user-images.githubusercontent.com/90487843/151362362-18e7a033-434d-47ee-819d-349c8446e5b6.png)

### 테마 자연어 유사도
- content : keyword와 category_group_name를 활용한 말뭉치
  - 키워드 : '맛집', '분위기 좋은', '테마파크', '오션뷰', '감성', '가족여행', '체험', '휴식', '레포츠', '가볼만한 곳'
  - 카테고리 : '관광명소', '음식점', '카페'
- CountVectorizer를 통해 벡터화
![image](https://user-images.githubusercontent.com/90487843/151370484-9c07b0fc-3f35-43bf-99a4-c775f06ad850.png)
- Cosine Similarity
  - 코사인 유사도 matrix
![image](https://user-images.githubusercontent.com/90487843/151370520-57e9fad6-f013-4cc2-9459-d321c510e00f.png)
  - 코사인 유사도 적용 예시
    ![image](https://user-images.githubusercontent.com/90487843/151370568-f0f2c85d-145e-4503-b839-947965d55df2.png)

### 경로 추천
- 카카오내비 API : 장소 좌표를 파라미터로 사용 -> 경로 추천
```python
# 헤더
headers = {"Authorization" : "KakaoAK {}".format(rest_api_key)}
# 파라미터
url1 = "https://apis-navi.kakaomobility.com/v1/directions?origin={경도1},{위도1}&destination={경도2},{위도2}&waypoints={경도,위도|...}"
url2 = "https://apis-navi.kakaomobility.com/v1/directions?origin={경도3},{위도3}&destination={경도4},{위도4}&waypoints={경도,위도|...}"

# GET을 이용하여 획득
res1 = requests.get(url1, headers=headers)
# Json을 이용하여 해제
doc1 = json.loads(res1.text)

res2 = requests.get(url2, headers=headers)
doc2 = json.loads(res2.text)
```
  - 경로 성공하는 경우
![image](https://user-images.githubusercontent.com/90487843/151372457-35ab1281-c370-425e-b1c5-9c3ea34a3000.png)
    - Folium을 사용하여 경로 시각화  
![image](https://user-images.githubusercontent.com/90487843/151374783-8df2c07a-f614-4f13-b795-83015773993c.png)
  - 경로 찾기 실패하는 경우
    ![image](https://user-images.githubusercontent.com/90487843/151372513-8beff087-520a-4838-887c-52a439c56f86.png)
- **경로 탐색이 실패하는 경우를 위해 직선 거리 경로도 추천**

## 4. 최종 시스템
![image](https://user-images.githubusercontent.com/90487843/151375417-f1773826-cd5c-47ab-badd-4aec642b532d.png)
- 구현 조건
  - 출발 지점 : 제주 공항
  - 장소 선택 : 이전 장소 15KM내에 있는 장소 중 선택(위도와 경도를 사용한 직선거리 사용)
  - 관광명소 : 이미지 유사도, 자연어 유사도, 평점 활용
  - 음식점 : CLIP 라벨링, 자연어 유사도, 평점 활용
  - 카페 : 자연어 유사도, 평점 활용
### 시스템 동작 예시
#### 1) 키워드 선택
![image](https://user-images.githubusercontent.com/90487843/151379030-6c33f464-7eef-4603-a938-e826c536a5ff.png)
#### 2) 카테고리 선택
![image](https://user-images.githubusercontent.com/90487843/151379085-0e805cc2-9ec6-45ce-94e5-6695dfbf087b.png)
#### 3) 대표 장소 선택(카페일 경우 실행X)
![image](https://user-images.githubusercontent.com/90487843/151379209-28219f46-a738-4be5-b760-2376b005d4c4.png)
![image](https://user-images.githubusercontent.com/90487843/151379254-191d8ffc-d6ec-4804-a50a-71a4f449a53a.png)
#### 4) 관광지 추천
- 대표 장소를 기준으로 관광지 추천 진행
- 카테고리, 대표 장소 선택 반복
  - 종료 : 카테고리 선택시 0 입력
#### 5) 경로 탐색
- 카카오내비 API 활용 경로 탐색
- 직선 거리 기준 경로 탐색
#### 6) 경로 출력
- 탐색한 경로 결과를 **folium**으로 시각화
- 관광지 주변 3KM 이내의 숙박 업소 표시
- 카카오내비 API 경로  
![image](https://user-images.githubusercontent.com/90487843/151382048-d5eb9d7f-c9db-4b0f-b590-a980df8d2375.png)
- 직선 거리 기준 경로  
![image](https://user-images.githubusercontent.com/90487843/151382095-ce4f8c77-fa95-4c1b-89c2-32e0973c6a8d.png)

## 5. 한계 및 보완점
#### 🛠사용 데이터 부족
- 추천에 사용한 주요 데이터는 이미지, 자연어, 평점 3가지이다.
  - 이미지
    - 음식점과 카페의 이미지는 **음식, 음료 등으로 특징 벡터로 분류하기 알맞지않다.**
    - 음식점의 경우 CLIP을 활용해 라벨링을 진행하였지만, 카페는 이미지를 사용한 작업을 진행하지 못하였다.
  - 자연어
    - 자연어 말뭉치를 keyword와 category_group_name 2가지를 조합하여 구성하였다.
    - 각 장소의 keyword는 대부분 1개이며, 최대 3개로 구성되어 **조합이 간단**하다.
    - 간단한 조합이기 때문에 코사인 유사도가 비슷한 경우가 많이 존재한다.
  - 평점
    - 장소별 페이지(place_url)에서 평점을 그대로 추출하여 사용한다.
    - **1명이 평가한 5점과 100명이 평가한 5점이 똑같은 평점으로 인식된다.**
- 획득한 데이터에서 여러 제한점이 발견되어 추천 알고리즘을 간단한 형식으로만 구성할 수 있었습니다.
- 평점 뿐아니라 사람들의 리뷰를 활용하면 조금 더 보완할 수 있을 것이라 생각된다.
#### 🛠결과의 평가지표 부재
- 라벨이 존재하는 지도 학습이 아니기 때문에 추천 결과에 대한 평가지표가 없다.

## 6. 참고 자료
- 카카오 키워드 검색 API : https://developers.kakao.com/docs/latest/ko/local/dev-guide
- 카카오내비 API : https://developers.kakao.com/docs/latest/ko/kakaonavi/common
- 이미지 특징 추출 : https://blog.naver.com/PostView.nhn?blogId=syg7949&logNo=221883713196&categoryNo=27&parentCategoryNo=27&from=thumbnailList
- CLIP Doc : https://openai.com/blog/clip/
- CLIP Git : https://github.com/openai/CLIP

## 7. 구성 인원
- 김준형 : <a href = 'https://github.com/Kkamz' target='_blink'>Github</a>
- 김남규 : <a href = 'https://github.com/Isanghada' target='_blink'>Github</a>

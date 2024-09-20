# FINAL_project
## Introduction
언어 교육 어플리케이션 리뷰(VoC) 분석 (스픽 vs. 말해보카)

## 프로젝트 개요
1. 팀원: 강민지, 김예림, 김이영
2. 수행기간: 2024.08.19 ~ 2024.09.09

## 프로젝트 진행 순서
![Blank diagram (2)](https://github.com/user-attachments/assets/c896bade-231e-4602-a17b-fb2f94de09ce)


## 주제 소개 및 선정 배경

- 개인 맞춤형 학습과 즉각적인 피드백을 제공하는 AI 기반 영어 학습 앱의 부상
- 효율적인 영어 공부 + 모바일 앱의 편리함 → 수요 증가
- 사용자 리뷰 분석을 통해 앱의 장점과 개선점 파악

## 데이터 수집

분석 대상 앱 : 스픽, 말해보카

기간 : 2020/01/01~2024/08/25

출처 : google playstore

추출 피쳐 : 
- **content(리뷰)**
- **score(평점)**
- **thumbsUpCount(리뷰 공감 수)**
- **at(날짜)**
- **appVersion**

## 데이터 전처리

- 이모지 및 특수문자 삭제
- 영어 대/소문자를 소문자로 통일
- py-hanspell: 네이버 맞춤법 검사 API를 활용하여 맞춤법 검사 및 띄어쓰기 조정
- PyKoSpacing: 추가로 띄어쓰기 조정

1. 형태소 분석기 선택 (Okt, Bareun), 불용어 처리
    - 형태소 분석기 비교
        - Okt: 단어를 덩어리 단위로 비교적 크게 분해함
        - 바른: 단어를 형태소 단위로 비교적 잘게 쪼갬
        - → 형태소 분석을 더 세밀하게 하는 바른 형태소 분석기가 한국어에 더욱 적합할 것이라 판단
    - 불용어 처리
        - 관형사, 조사, 어미 등과 같은 품사 제거
            - ex) 이, 그, 저, 을, 를, 요
        - 영어 학습 어플리케이션 특성상 자주 언급되지만 분석에 영향 주지 않는 단어 제거
            - ex) 영어, 어플, 앱
        - 의미를 알아볼 수 없는 오타 제거
2. 반지도학습 선정 이유, 데이터 스플릿 (클래스 분포 확인)
    - 반지도학습 선정 이유
        - 단순히 별점만으로 리뷰의 긍정/부정/중립을 구분하기 어려움
          ![image](https://github.com/user-attachments/assets/7933b2de-887b-42ad-a695-86bee5bd9f50)
        - 가용할 수 있는 리소스에 한계가 있으므로 모든 데이터를 수동으로 라벨링할 수 없음
         → 일부는 **수동**으로, 일부는 **자동**으로 라벨링하는 반지도학습을 선정
    - 데이터 스플릿
        - 총 데이터를 9:1로 나눠 Train, Test Set으로 구분
        - Train Set을 다시 8:2로 나눠 Auto Train, Manual Train으로 구분
        - Manual Train, Test Set만 수동 라벨링을 진행했으며, Auto Train은 수동 라벨링 결과를 바탕으로 자동 라벨링을 실시
        ![image](https://github.com/user-attachments/assets/6a777668-6c7b-4ebc-b2a9-bb33bf4016e8)

## 감성 분석

1. 감성 사전
- 형태소 및 품사에 대한 극성을 모두 고려하여 긍정, 부정, 중립으로 감성사전 편찬
- 감성사전을 활용하여 Test Set에 대해 감성 분석 진행
- 긍정, 부정 리뷰도 중립으로 분류하는 경향
<p align="center">
<img src="https://github.com/user-attachments/assets/508924a3-0037-4348-9090-894f97c9c74a" width="600" height="225">

<p align="center">
  <img src="https://github.com/user-attachments/assets/4c5f47fe-601a-4852-b536-051c7da00c47" alt="감성사전_score">
</p>


2. TF/IDF
- 진행 순서 : Logistic Regression → 오버샘플링/언더샘플링 → Random Forest → XGBoost
- 최적 파라미터를 적용한 Logistic Regression의 성능이 가장 우수함
    - 최적 파라미터 : class_weight = {1.0: 1, -1.0: 3, 0.0: 7}
- 감성사전을 활용했을 때보다 성능이 전반적으로 좋아졌지만 중립 클래스의 성능이 여전히 좋지 않음
<p align="center">
<img src="https://github.com/user-attachments/assets/94d1533f-1fbb-47e3-8949-ca8532fa7c3c" width="600" height="225">


**Logistic Regression Classification Report**
<p align="center">
  <img src="https://github.com/user-attachments/assets/8602c0e3-7205-45a5-926f-039762d67d80" alt="tfidf_score">
</p>


3. KoBERT
- SKTBrain에서 개발한 BERT 기반의 사전 학습 모델로, 양방향으로 문맥을 이해할 수 있어 한국어 리뷰의 감성 분석에 적합함
- Manual Train Set을 하이퍼 파라미터 튜닝
    - Stratified K-Fold를 사용하여 어플리케이션 별 비율을 유지
    - 최적 파라미터: learning rate=0.00002, batch size = 8, num_epochs=4, weight_decay =1
- 최적 모델로 Auto Train Set에 대해 Fine Tuning 후 Test Set 예측
- 감성사전, TF/IDF를 이용한 감성 분석보다 좋은 성능을 보여줌
<p align="center">
<img src="https://github.com/user-attachments/assets/18aeb0ea-1d22-44e8-ab17-d8eb229d7b9f" width="600" height="225">


<p align="center">
  <img src="https://github.com/user-attachments/assets/8ec6a155-e5d1-4686-9fa4-da8a6cec558d" alt="kobert_score">
</p>

## LDA 토픽 모델링

- 문서 내에서 잠재적인 주제를 추출하는 기법으로, 리뷰 데이터의 주요 주제를 파악하기 위해 사용
- Topic 수는 3개로 선정
- 전체 리뷰: 긍정, 중립, 부정 전체 데이터를 분석
- 부정적인 리뷰: 부정적인 리뷰만 따로 추출하여 분석

1. 스픽
<p align="center">
  <img src="https://github.com/user-attachments/assets/bff7cba0-4768-4248-978d-ee0a4139f37f" alt="image1" width="800">
</p>

- 전체 리뷰: 유저들은 결제 및 서비스와 관련된 경험을 가장 많이 언급, 학습 과정과 경험에 대한  피드백도 자주 등장

- 부정적인 리뷰: 유저들 대부분이 음성 인식, 결제 및 구독, 오류 및 사용자 경험에 대한 불만을 주로 언급

2. 말해보카
<p align="center">
  <img src="https://github.com/user-attachments/assets/f5601167-8398-40aa-9fa5-7a5c4b9b2591" alt="image2" width="800">
</p>

- 전체 리뷰: 유저들이 어휘 학습과 관련된 내용을 가장 많이 언급, 학습 경험에 대한 긍정적인 평가가 있었지만 결제 및 서비스 관련 부정 피드백도 자주 등장

- 부정적인 리뷰: 어휘 학습과 관련된 피드백이 가장 많이 등장, 그 중 음성인식에 대한 문제가 많이 등장. 결제 이슈와 서비스 이용과 가격의 문제도 자주 등장

3. 스픽 & 말해보카 공통 불만
<div align="center">
    
| 공통 불만 |
|:---:|
| (1) 음성 인식 오류 및 학습 기능 |
| (2) 결제/환불 과정 |
| (3) 광고 및 유료 서비스 |

</div>

## EDA
<스픽>
![스크린샷 2024-09-19 132525](https://github.com/user-attachments/assets/289c1fd9-51e2-46df-b95c-44d3f4a64998)

<div align="center">
    
| 스픽 월별 감성 분 |  상위 10개 앱 버전별 감성 분포 |
| --- | --- |
| - 2024년 1월 리뷰수가 가장 많음  <br> - 2024년 1월에는 대대적인 서비스 개편을 단행했고,<br> '스픽 튜터'라는 새로운 기능을 통해 개인화된 학습 경험 제공 | - 3.1 그룹에 가장 많은 리뷰가 있음 <br> - 3.13.4 버전이 리뷰 수가 가장 많고,<br> 긍정적인 리뷰 비율도 높음 |

</div>

![스크린샷 2024-09-19 133329](https://github.com/user-attachments/assets/bf0beffb-1371-49d8-9099-d9946fff073b)

<div align="center">
    
| 점수와 감성 레이블 간 상관관 |
| --- |
| - 점수가 낮을수록 부정적인 감정을 많이 담고 있음  <br> - 점수가 높을수록 긍정적인 감정을 많이 담고 있음 <br>     = 점수가 높을수록 긍정적인 리뷰가 많고, 점수가 낮을수록 부정적인 리뷰가 많음 |

</div>

![스크린샷 2024-09-19 133439](https://github.com/user-attachments/assets/d8707c05-6ffc-4aec-897f-0dbec7db4efb)

<div align="center">
    
| 앱 버전별 평균 점수 |  시간에 따른 평균 점수 |
| --- | --- |
| - 전반적으로 4점대 중반에서 5점에 가까운 높은 점수를 유지  <br> - 몇 몇 버전에서 점수가 급격히 하락 | - 2020년 말 점수가 크게 하락했다가 다시 회복 <br> - 이후로는 4점대 초반에서 중반을 유지 <br> - 2024년 다시 회복 추세를 보임 |

</div>

![스크린샷 2024-09-19 133611](https://github.com/user-attachments/assets/e7e6d968-1177-483c-aa40-e6a84f6a916a)

<div align="center">
    
| 감성 레이블 평균 리뷰 공감 수 |  점수별 평균 리뷰 공감 수 |
| --- | --- |
| - 부정적인 감성 레이블이 가장 많은 리뷰 공감 수를 받음  <br> - 평균적으로 부정적인 리뷰는 3개 이상의 리뷰 공감 수를 받음  <br> - 부정적인 리뷰가 더 많은 사용자들의 공감을 얻음을 확인할 수 있음 | - 1점&2점 리뷰가 가장 많은 리뷰 공감 수를 받음 <br> - 부정적인 점수를 준 리뷰가 다른 사용자들의 공감을 얻음을 확인 |

</div>

<말해보카>
![스크린샷 2024-09-19 134112](https://github.com/user-attachments/assets/0fff9b72-bef7-4153-bf51-754d3994693f)
![스크린샷 2024-09-19 134254](https://github.com/user-attachments/assets/40e9711a-b811-4247-838e-6570166fcd0c)
![스크린샷 2024-09-19 134347](https://github.com/user-attachments/assets/5b61c42b-bc16-4eff-ab3d-25c0d1861f93)
![스크린샷 2024-09-19 133857](https://github.com/user-attachments/assets/45fd2453-98c2-4445-b658-df98ce9e1301)



## 비즈니스 효과

감성 분석과 토픽 모델링을 함께 사용함으로써 사용자들이 어떤 부분에서 만족하고,
불만을 느꼈는지 빠르게 확인 및 조치할 수 있음

1. 긍정적 피드백 분석의 효과
- 사람들이 긍정적으로 평가한 기능 및 요소를 강화 가능
- 긍정적 피드백을 마케팅 진행 시 강조할 수 있음
  
2. 부정적 피드백 분석의 효과
- 사용자들이 반복적으로 언급하는 문제에 대해 제품 개선점 파악 ⇒ 고객 이탈 방지
- 새로운 기능 추가 가능 ⇒ 신규 고객 확보

## 추후 발전 방안

- 목표: 데이터 파이프라인 및 대시보드 구축
1. **데이터 파이프라인 구축**
- 실시간으로 고객 리뷰를 크롤링하여 DB에 적재
- 성과가 가장 좋은 KoBERT 모델을 이용하여 실시간으로 리뷰를 감성 분석
  
2. **대시보드 구축**
- 고객 리뷰를 실시간으로 감성 분석, 토픽별 구분
- 대시보드에 즉각 반영 -> 시각화

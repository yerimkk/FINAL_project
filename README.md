# FINAL_project

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
![image](https://github.com/user-attachments/assets/508924a3-0037-4348-9090-894f97c9c74a)
![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/4624b43e-7f6d-41f9-87ce-569ebe6e1301/fe37dd2b-a39b-4e6f-8bc7-1b4d9da300aa/image.png)

2. TF/IDF

3. KoBERT
- SKTBrain에서 개발한 BERT 기반의 사전 학습 모델로, 양방향으로 문맥을 이해할 수 있어 한국어 리뷰의 감성 분석에 적합함
- Manual Train Set을 하이퍼 파라미터 튜닝
    - Stratified K-Fold를 사용하여 어플리케이션 별 비율을 유지
    - 최적 파라미터: learning rate=0.00002, batch size = 8, num_epochs=4, weight_decay =1
- 최적 모델로 Auto Train Set에 대해 Fine Tuning 후 Test Set 예측
- 감성사전, TF/IDF를 이용한 감성 분석보다 좋은 성능을 보여줌
![image](https://github.com/user-attachments/assets/38aec6f5-3b57-4a9f-a590-55932d188df0)
![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/4624b43e-7f6d-41f9-87ce-569ebe6e1301/cf61c657-1a7f-4b52-b268-23aa9cd5880e/image.png)



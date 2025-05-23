# 영화 리뷰 분석 샘플 제작 계획서

## 1. 목표
`prd_attention_review_interpretation.md` 문서에 기술된 "통합적 주의 기반 리뷰 해석 모델"의 핵심 아이디어를 검증하기 위한 분석 샘플을 제작한다. 구체적으로 영화의 줄거리, 포스터 (텍스트 캡션), 사용자 리뷰를 통합 입력으로 사용하여 어텐션 메커니즘을 통해 요소 간의 관계를 살펴보는 프로토타입을 개발한다.

## 2. 활용 데이터셋

### 2.1. 주 데이터셋: 네이버 영화 리뷰 (NSMC)
- **URL**: [https://github.com/e9t/nsmc](https://github.com/e9t/nsmc)
- **내용**: 영화 ID, 한국어 리뷰 텍스트, 긍/부정 레이블
- **활용**: 사용자 리뷰 데이터로 활용한다.

### 2.2. 추가 정보 소스: KMDB (한국영화데이터베이스) API
- **URL**: [https://www.kmdb.or.kr/info/api/apiDetail/6](https://www.kmdb.or.kr/info/api/apiDetail/6) (또는 유사 영화 정보 API)
- **내용**: 영화 제목, 감독, 배우, 줄거리, 포스터 URL, 관련 이미지 등
- **활용**: 영화별 줄거리 정보 및 포스터 캡션(포스터 이미지에 대한 텍스트 설명이 없다면, 포스터 이미지 URL을 기반으로 주요 특징을 간략히 기술하거나, 영화 관련 텍스트 정보로 대체)을 수집한다.

## 3. 분석 단계 (PRD 기능 요구사항 기반)

### 3.1. 입력 전처리 (PRD F1)
- **F1.1**: NSMC 데이터셋에서 영화 리뷰를 로드한다. KMDB API를 사용하여 매칭되는 영화의 줄거리 및 포스터 관련 텍스트 정보를 가져온다.
    - 포스터 캡션의 경우, API에서 직접 제공하지 않으면 포스터 이미지의 주요 특징을 간략히 기술하거나, 영화 소개 문구 등으로 대체한다.
- **F1.2**: 모든 텍스트(줄거리, 포스터 캡션, 리뷰)에 대해 일관된 토크나이저(예: KoBERT 또는 HuggingFace의 BERT 호환 한국어 토크나이저)를 적용한다.
- **F1.3**: PRD 샘플 입력 형식에 따라 `[CLS] 줄거리 [SEP] 포스터_캡션 [SEP] 리뷰_1 [SEP] 리뷰_2 [SEP] ...` 형태로 통합된 시퀀스를 구성한다.

### 3.2. 모델 활용 (PRD F2)
- **F2.1**: 사전 학습된 트랜스포머 모델(예: KoBERT 또는 HuggingFace 라이브러리의 BERT 계열 모델)을 로드한다.
- **F2.2**: 통합된 입력 시퀀스에 대해 모델의 셀프 어텐션(self-attention) 메커니즘을 통과시켜 어텐션 가중치를 얻는다. (별도의 학습 과정 없이, 사전 학습된 모델의 어텐션 값만 활용)
- **F2.3**: 본 샘플에서는 별도의 지도 학습용 헤드는 구성하지 않고, 어텐션 맵 해석에 집중한다.

### 3.3. 해석 및 시각화 (PRD F4)
- **F4.1**: 특정 어텐션 헤드 및 레이어에서 어텐션 매트릭스를 추출한다.
- **F4.2**: 각 리뷰 토큰들이 영화 줄거리의 어떤 부분, 포스터 캡션의 어떤 부분에 주목하는지 어텐션 가중치를 기반으로 계산한다.
- **F4.3**: (선택 사항) 리뷰별로 줄거리나 포스터 캡션 내 중요 키워드/구문을 어텐션 가중치 합계를 기준으로 순위화한다.
- **F4.4**: 간단한 시각화 방법을 구현한다.
    - 예: 특정 리뷰와 줄거리/포스터 간의 어텐션 강도를 텍스트 하이라이팅 또는 matplotlib을 이용한 간이 히트맵으로 표현한다.
- **F4.5**: (선택 사항) 리뷰 간 어텐션 관계는 본 샘플 범위에서는 제외하거나, 매우 간략하게만 언급한다.

## 4. 예상 산출물
- 데이터 수집 및 전처리 파이썬 스크립트 (`.py` 또는 `.ipynb`)
- 통합 입력 생성 및 BERT 모델 통과, 어텐션 추출 파이썬 스크립트
- 1~2개 영화에 대한 샘플 분석 결과:
    - 입력 텍스트 시퀀스 예시
    - 선택된 리뷰와 줄거리/포스터 간의 어텐션 시각화 (간단한 이미지 또는 텍스트)

## 5. 간략 일정
- **1주차**:
    - NSMC 데이터셋 다운로드 및 구조 파악
    - KMDB API 사용 승인 및 테스트, 필요한 영화 정보(줄거리, 포스터 관련 텍스트) 수집 함수 개발
    - 샘플 영화 선정 및 데이터 통합 (리뷰 + 줄거리 + 포스터 캡션)
    - 토크나이저 선정 및 통합 입력 시퀀스 생성 스크립트 초안 작성
- **2주차**:
    - 사전 학습된 BERT 모델 로드 및 어텐션 매트릭스 추출 기능 구현
    - 리뷰-줄거리, 리뷰-포스터 간 어텐션 계산 로직 개발
    - 기본적인 어텐션 시각화 방법 구현 (텍스트 하이라이트 또는 matplotlib 기반)
    - 선택된 샘플 영화에 대한 분석 결과 정리 및 계획서 업데이트

## 6. 참고사항
- 포스터 캡션의 경우, API에서 명시적으로 제공하지 않는다면 영화의 핵심적인 시각적 요소나 분위기를 나타내는 짧은 텍스트로 대체하거나 생성하는 방안을 고려한다.
- 초기 샘플 제작 단계에서는 모델의 미세 조정(fine-tuning)은 진행하지 않고, 사전 학습된 모델의 어텐션 패턴을 관찰하는 데 집중한다.

---
_본 계획은 초기 구상이며, 진행 상황에 따라 변경될 수 있습니다._ 
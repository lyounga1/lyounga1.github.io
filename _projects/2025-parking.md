---
layout: page
title: 실시간 주차 공간 확인 시스템(HOWPARKING)
description: AI분석 기반 주차자리 찾기 서비스
img: assets/img/howparking_logo.png
importance: 1
category: project
---

## 👨‍💻 프로젝트 역할 (Contribution)
**Role: AI & Backend Developer**
- **FastAPI**와 **Discord Bot**을 연동하여 실시간 AI 분석 자동화 파이프라인을 구축했습니다.
- **설치된 카메라가 일정 시간마다 이미지를 전송하면** **YOLOv8** 모델이 이를 즉시 분석하고, 그 결과(시각화된 이미지 및 JSON 데이터)를 채널에 자동으로 반환하는 시스템을 구현했습니다.
---

## 💡 프로젝트 개요
주차난 해소를 위해 개발한 **'실시간 주차 공간 확인 시스템'**입니다.
설치된 카메라가 촬영한 주차장 이미지를 분석하여, 사용자가 웹사이트와 디스코드 서버에서 빈자리를 바로 확인할 수 있도록 돕습니다.

별도의 복잡한 서버 구축 대신, **디스코드(Discord) 봇**을 활용하여 이미지를 주고받고 분석 결과를 처리하는 효율적인 방식을 고안했습니다.

### 🏗️ 시스템 동작 원리 (Work Flow)

1. **촬영 및 업로드:** 주차장 카메라가 사진을 찍어 디스코드 채널에 올립니다.
2. **봇의 실시간 감지 (My Part):**
   - 제가 만든 **디스코드 봇**이 채널을 24시간 감시하다가 새 사진이 올라오면 즉시 가져옵니다.
   - 봇 내부의 **AI 모델(YOLOv8)**이 주차된 차량과 빈 공간을 분석합니다.
3. **결과 전송 (My Part):** 봇이 분석된 결과(빈자리 개수, 위치 정보)와 처리된 이미지를 다시 채널에 업로드합니다.
4. **웹사이트 표시:** 팀원의 웹사이트가 이 정보를 가져와 화면에 보여줍니다.

---

## 🛠️ 핵심 기술 및 구현 과정

### 1. YOLOv8 (Object Detection AI)
- **주차 공간 맞춤형 객체 탐지:** : 주차장 환경에 특화된 데이터셋으로 YOLOv8 모델을 파인튜닝(Fine-tuning)하여 차량 인식률을 상승시켰습니다.
- **실시간 점유 상태 판별:** 객체 탐지 결과(Bounding Box)와 주차 슬롯 좌표(Polygon)를 매핑하는 알고리즘을 적용하여, 각 주차면의 점유/빈자리 여부를 실시간으로 판별했습니다.
- **모델 최적화 및 트러블슈팅:**
  정확도와 속도 간의 균형을 맞추기 위해 단계적인 모델 교체를 수행했습니다.
  * **`yolov8n` → `yolov8s`:** 초기엔 속도가 빠른 Nano 모델을 썼으나 원거리 인식률이 낮아, Small 모델로 교체해 정확도를 상승시켰습니다..
  * **`yolov8x` 최종 도입:** "햇빛 반사 시 **흰색 차량**이 주차선과 겹쳐 인식되지 않는 현상을 해결하기 위해, 가장 강력한 Extra Large 모델을 도입하여 미인식 문제를 해결했습니다.

### 2. FastAPI
- **서버 구축:** FastAPI를 사용하여 다수의 이미지 분석 요청을 병목 없이 처리하는 고성능 백엔드 서버를 구축했습니다.
- **RESTful API 설계:** 이미지 데이터를 받아 분석 결과(감지된 객체 수, 좌표, 처리된 이미지)를 JSON으로 반환하는 API를 구현했습니다.


### 3. Discord Bot을 활용한 데이터 파이프라인 자동화
- **실시간 이미지 감지 및 자동화::** **'수집 전용 채널'**에 사진을 올리면, 봇이 이를 감지해서 AI 서버로 보냅니다. 사람이 일일이 확인하지 않아도 24시간 돌아갈수있게 따로 배포를 하였습니다.
- **시스템 연결:** 봇을 **[카메라]**와 [AI 분석 서버] 사이를 이어주는 '다리' 역할을 하도록 개발하여 데이터가 끊김 없이 흐르도록 만들었습니다.
--

## 💻 실행 결과

### 1. 이미지 수집 자동화 (Input Channel)
설치된 카메라(Client)가 영상을 캡처하여 디스코드 **수집 전용 채널**에 업로드한 모습입니다.
'Image Bot'이 영상을 감지하는 순간, 백그라운드에서 제 파이프라인이 작동을 시작합니다.

<div class="row justify-content-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/parking_input.png" title="CCTV 이미지 수집" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ▲ <strong>수집 채널:</strong> 카메라가 촬영한 원본 이미지가 실시간으로 전송된 모습
</div>

### 2. AI 분석 및 결과 알림 (Result Channel)
FastAPI 서버에서 YOLOv8 분석이 완료되면, **결과 알림 채널**에 분석 리포트가 도착합니다.
단순 이미지뿐만 아니라 **전체 주차면 수, 현재 주차된 차량 수, 빈 공간 비율** 등의 데이터가 함께 제공됩니다.

<div class="row justify-content-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/parking_result.png" title="AI 분석 결과" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ▲ <strong>결과 채널:</strong> AI가 차량(Red)과 빈자리(Green)를 구분하여 시각화한 분석 결과
</div>

### 3. 최종 웹 서비스 연동 (Web Dashboard)
디스코드 봇이 처리한 데이터는 최종적으로 **사용자용 웹사이트**에 반영됩니다.
사용자는 지도를 통해 주차장 위치를 확인하고, 팝업창을 통해 실시간 주차 현황(여유/혼잡)을 직관적으로 파악할 수 있습니다.

<div class="row justify-content-center">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/parking_web.png" title="웹 대시보드" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ▲ <strong>웹 대시보드:</strong> 봇이 분석한 데이터를 받아 사용자에게 제공되는 최종 서비스 화면
</div>

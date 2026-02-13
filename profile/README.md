# 🏋️‍♂️ LevelUpFit: Smart AI Fitness Platform

**LevelUpFit**은 AI 기술을 활용하여 사용자의 운동 자세를 정밀하게 분석하고, 체계적인 루틴 관리를 지원하는 스마트 피트니스 플랫폼입니다. 부상 방지와 효율적인 운동 성장을 돕기 위해 실시간 피드백과 데이터를 제공합니다.

---

## 📸 Main Image
<img width="3150" height="1330" alt="대표 이미지  AI 기반 자세 분석 헬스케어 서비스_LevelUpFit" src="https://github.com/user-attachments/assets/a35a7662-9af1-4d81-ab6d-27589e1cf1af" />

## 🌟 Core Projects

우리 조직은 최상의 피트니스 경험을 제공하기 위해 다음과 같은 독립된 모듈로 구성되어 유기적으로 작동합니다.

### 1. [Main Backend](https://github.com/LevelUpFit/main-backend) ⚙️
플랫폼의 중추 역할을 담당하는 서비스로, 사용자 경험의 전반을 관리합니다.
- **주요 기능**: JWT 기반 인증(Form & Kakao), 운동 종목 및 루틴 관리, 사용자 운동 이력 저장 및 조회.
- **Tech Stack**: Spring Boot, JDK 17, Gradle.

### 2. [Pose Backend](https://github.com/LevelUpFit/pose-backend) 🧠
AI 모델을 사용하여 운동 영상을 분석하고 정밀한 데이터를 산출합니다.
- **주요 기능**: MediaPipe 기반 포즈 추정, 런지/스쿼트 자세 분석(무릎-발끝 관계, 가동범위, 수직 정렬), 수축/이완 시간 산출 및 시각화 피드백 제공.
- **Tech Stack**: FastAPI, Python, MediaPipe, MinIO.

| Good | Bad |
| :--- | :--- |
| ![분석영상-good](https://github.com/user-attachments/assets/adadc8b6-4782-4788-9fd0-50b238cb8c0e)| ![분석영상-bad](https://github.com/user-attachments/assets/b5fbfcaf-0449-4cc6-9de6-c55907df1799)|

### 3. [Frontend](https://github.com/LevelUpFit/front) 🖥️
사용자가 서비스를 이용하는 인터페이스로, 직관적이고 반응성이 뛰어난 환경을 제공합니다.
- **주요 기능**: 실시간 AI 자세 분석 화면 제공, 운동 루틴 생성 및 관리, 개인별 운동 대시보드.
- **Tech Stack**: React, Vite, TailwindCSS, Recoil.

---

## 🛠 Tech Stack Overview

| Category | technologies |
| :--- | :--- |
| **Language** | Java (JDK 17), Python, JavaScript (ES6+) |
| **Framework** | Spring Boot, FastAPI, React |
| **AI/Analysis** | MediaPipe Pose Model |
| **Storage** | MinIO (Object Storage) |
| **Style/Auth** | TailwindCSS, JWT (OAuth2 - Kakao) |

---

## 🏗 System Architecture
<img width="954" height="560" alt="수정 2  개발구성도" src="https://github.com/user-attachments/assets/cffa1401-d485-44e7-b2c3-a29aad07607f" />


Main Server: 사용자 관리 및 루틴 도메인 로직 담당 (RDBMS 활용)

AI Server: MediaPipe 기반 비동기 영상 분석 및 좌표 추출 (FastAPI)

Storage: 분석 전/후 영상 및 데이터 보존 (MinIO)

---

# 🔥 Key Challenges & Solutions (Technical Deep Dive)

### 1. 이기종 서버 간의 비동기 Multipart 데이터 통신 최적화
* **Challenge**: Spring Boot 서버에서 관리하는 비즈니스 메타데이터와 사용자가 업로드한 대용량 영상 파일을 Python AI 서버로 전송할 때, 파일 데이터가 유실되거나 동기식 처리로 인한 스레드 차단 병목 현상이 발생할 위험이 있었습니다.
* **Solution**: `WebClient`를 활용한 논블로킹 통신을 구축하고, `MultipartBodyBuilder`를 통해 데이터를 구조화했습니다. 특히 `ByteArrayResource`를 상속받아 `getFilename()`을 오버라이드함으로써 멀티파트 전송 시 파일명이 Null로 처리되는 문제를 해결하고, AI 분석 결과(`FeedbackresultDTO`)를 비동기 Mono 객체로 수신하여 자원 효율성을 극대화했습니다.

```java
// FastApiWebClientService.java
public Mono<FeedbackresultDTO> sendToFastApi(ExerciseFeedbackRequest request) throws IOException {
    MultipartBodyBuilder builder = new MultipartBodyBuilder();
    MultipartFile video = request.getVideo();
    byte[] bytes = video.getBytes();

    // 파일명 유실 방지를 위한 Resource 커스텀 구현
    ByteArrayResource videoResource = new ByteArrayResource(bytes) {
        @Override
        public String getFilename() {
            return video.getOriginalFilename();
        }
    };
    
    builder.part("file", videoResource)
            .header("Content-Disposition", "form-data; name=file; filename=" + video.getOriginalFilename());
    builder.part("exercise_id", request.getExerciseId());
    builder.part("feedback_id", request.getFeedbackId());

    return webClient.post()
            .uri(poseUrl)
            .contentType(MediaType.MULTIPART_FORM_DATA)
            .bodyValue(builder.build())
            .retrieve()
            .bodyToMono(FeedbackresultDTO.class);
}
```

---

### 2. WebSocket 기반 실시간 알림 및 세션 생명주기 관리
* **Challenge**: AI 영상 분석은 고부하 작업으로 수 초 이상의 시간이 소요되므로, 사용자가 분석 완료 시점을 실시간으로 인지하지 못하면 서비스 이탈이나 반복적인 요청이 발생할 수 있는 UX적 한계가 있었습니다.
* **Solution**: `ConcurrentHashMap`을 이용해 세션을 ID 단위로 관리하는 `FeedbackWebSocketHandler`를 구현했습니다. 분석 서버로부터 결과를 수신한 즉시 해당 세션에 `FEEDBACK_ANALYSIS_COMPLETE` 메시지를 푸시하고, 전송 직후 세션을 명시적으로 종료(`session.close()`)하여 서버의 커넥션 유지 비용을 절감하고 실시간성을 확보했습니다.

```java
// FeedbackWebSocketHandler.java
public void sendAnalysisCompleteMessage(int feedbackId) {
    WebSocketSession session = sessionMap.get(feedbackId);
    if (session != null && session.isOpen()) {
        try {
            // 분석 완료 JSON 메시지 푸시
            String message = String.format("{\"type\": \"FEEDBACK_ANALYSIS_COMPLETE\", \"feedbackId\": %d}", feedbackId);
            session.sendMessage(new TextMessage(message));
            session.close(); // 자원 효율화를 위한 세션 즉시 종료
        } catch (IOException e) {
            logger.error("WebSocket 전송 실패: feedbackId={}", feedbackId, e);
        }
    }
}
```

---

### 3. MediaPipe 및 지수 함수 기반의 정밀 자세 평가 알고리즘
* **Challenge**: 관절 좌표 데이터만으로는 "올바른 자세"에 대한 객관적 기준을 수치화하기 어려웠으며, 미세한 떨림과 심각한 자세 붕괴를 구분할 변별력이 필요했습니다.
* **Solution**: `calc_penalty` 알고리즘에 지수 함수(Exponential Function)를 도입했습니다. 무릎이 발끝을 넘어서는 임계치(Threshold)를 기준으로 거리가 멀어질수록 패널티를 가중시켜, 자세가 불안정해질수록 정확도 점수가 급격히 하락하게 설계함으로써 분석 결과의 신뢰도를 높였습니다.

```python
# lunge_analyzer_level3.py
def calc_penalty(over_distance, threshold=10, max_penalty=100):
    x = max(0, over_distance)
    # 거리 증가에 따른 패널티 가중치 부여 (지수 함수 적용)
    exp_input = min((x / threshold) ** 2, 10)
    penalty = math.exp(exp_input) - 1
    return min(penalty, max_penalty)

# 앞다리 정렬 판별 및 패널티 산출
over_distance = max(0, knee_x - foot_x) if look_direction == "right" else max(0, foot_x - knee_x)
penalty = calc_penalty(over_distance) if over_distance > 0 else 0
```

---

### 4. FFmpeg를 활용한 웹 스트리밍 최적화 (faststart)
* **Challenge**: AI 분석 결과로 생성된 MP4 파일이 브라우저에서 스트리밍될 때, 메타데이터(moov atom)가 파일 끝에 위치하여 영상 전체가 다운로드되기 전까지 재생이 시작되지 않는 지연 현상이 발생했습니다.
* **Solution**: `FFmpeg` 라이브러리를 연동하여 인코딩 파이프라인을 구축했습니다. 특히 `-movflags +faststart` 옵션을 적용하여 동영상 헤더 정보를 파일 앞부분으로 배치함으로써, 네트워크 환경에 관계없이 사용자가 분석 영상을 즉시 확인할 수 있도록 스트리밍 성능을 개선했습니다.

```python
# lunge_analyzer_level3.py
try:
    # libx264 코덱 인코딩 및 스트리밍 최적화 플래그 적용
    subprocess.run([
        'ffmpeg', '-i', output_path,
        '-c:v', 'libx264',
        '-movflags', '+faststart', # 웹 재생 최적화의 핵심
        '-y',
        optimized_path
    ], check=True, capture_output=True)
except subprocess.CalledProcessError as e:
    print(f"FFmpeg 최적화 실패: {e.stderr.decode()}")
```

## 👥 Contributors

*우리 팀의 멋진 기여자들이 프로젝트를 이끌어가고 있습니다.*

<a href="https://github.com/LevelUpFit">
  <img src="https://github.com/LevelUpFit.png" width="100px;" alt="LevelUpFit Logo"/>
</a>

---
© 2024 LevelUpFit Team. All rights reserved.

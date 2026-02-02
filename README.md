# 🏋️‍♂️ LevelUpFit: Smart AI Fitness Platform

**LevelUpFit**은 AI 기술을 활용하여 사용자의 운동 자세를 정밀하게 분석하고, 체계적인 루틴 관리를 지원하는 스마트 피트니스 플랫폼입니다. 부상 방지와 효율적인 운동 성장을 돕기 위해 실시간 피드백과 데이터를 제공합니다.

---

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

LevelUpFit은 각 서비스 간의 효율적인 통신을 통해 데이터 흐름을 최적화했습니다.
1. **Frontend**에서 사용자가 운동 영상을 업로드합니다.
2. **Pose Backend**에서 영상을 프레임 단위로 분석하여 결과 데이터와 분석 영상을 생성합니다.
3. 생성된 결과는 **Main Backend**에 저장되어 사용자가 언제든 자신의 성장을 확인할 수 있습니다.

---

## 👥 Contributors

*우리 팀의 멋진 기여자들이 프로젝트를 이끌어가고 있습니다.*

<a href="https://github.com/LevelUpFit">
  <img src="https://github.com/LevelUpFit.png" width="100px;" alt="LevelUpFit Logo"/>
</a>

---
© 2024 LevelUpFit Team. All rights reserved.

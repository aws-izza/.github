<div align="center">

# IZZA — AI 기반 지능형 공간 데이터 분석 플랫폼

**데이터 기반 입지 분석, AI가 완성하는 기업 컨설팅**

[![AWS Cloud School](https://img.shields.io/badge/AWS_Cloud_School-9기-FF9900?style=flat-square&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![KDT Hackathon](https://img.shields.io/badge/제7회_KDT_해커톤-기술혁신상-FFD700?style=flat-square)](https://github.com/aws-izza)
[![Domain](https://img.shields.io/badge/Service-izza--nopizza.com-3b82f6?style=flat-square&logo=googlechrome&logoColor=white)](https://izza-nopizza.com)

</div>

---

## 프로젝트 소개

**IZZA**는 공공데이터포털과 V-World에서 수집한 **수천만 건의 토지 공간정보**를 기반으로, 기업의 입지 분석을 자동화하는 플랫폼입니다.

기업이 신규 공장·사업장 부지를 선정할 때, 토지면적·공시지가·용도지역·인프라 등 복합 조건을 설정하면 후보지를 탐색하고, **AWS Bedrock 기반 LLM 멀티 에이전트**가 투자 적합도를 분석한 구조화된 보고서를 자동 생성합니다.

> **기간:** 2025.07 — 2025.09 (약 3개월)  
> **주관:** 한국전파진흥협회, AWS 코리아 (AWS Cloud School 9기)  
> **수상:** 제7회 KDT 해커톤 기술혁신상 (고용노동부, 한국기술교육대학교 직업능력심사평가원)

---

## 핵심 기능

| 기능 | 설명 |
|------|------|
| **🔍 입지 후보지 탐색** | 용도지역·토지면적·공시지가·전기요금 등 조건 필터링, Kakao Map 기반 지도 시각화 |
| **📊 입지 비교 분석** | 업종별 맞춤 지표 선택 → 가중치 반영 → 후보지 간 종합 점수 산출 및 Ranking |
| **🤖 AI 보고서 자동 생성** | Strands Agents 멀티 에이전트가 입지조건·인프라·안정성을 분석하고 관련 정책까지 포함한 보고서 생성 |
| **📧 AI 뉴스레터 메일링** | 산업단지 고시 변경 자동 감지 → AI 분석 → 구독자에게 뉴스레터 자동 발송 |
| **🔎 검색어 자동완성** | Trie 자료구조 기반 In-Memory 처리로 50ms 이하 응답 |

---

## 아키텍처

```
[User] → Route 53 → CloudFront → WAF → ALB
                         ↓
                   Amplify (React)          ──→  Kakao Map API
                                            
         ┌──────── EKS Cluster (Private Subnet) ────────┐
         │                                               │
         │  ┌─────────────┐  ┌──────────────┐          │
         │  │ Spring Boot │  │     Gin      │          │
         │  │ (입지 검색  │  │ (자동완성)   │          │
         │  │  · 분석)    │  │ Trie+InMemory│          │
         │  └──────┬──────┘  └──────┬───────┘          │
         │         │                │                    │
         │  ┌──────┴────────────────┴───────┐           │
         │  │          FastAPI              │           │
         │  │      (AI 보고서 생성)          │           │
         │  │  ┌─────────────────────────┐  │           │
         │  │  │  Strands Agents SDK     │  │           │
         │  │  │  ┌───────────────────┐  │  │           │
         │  │  │  │ Orchestrator Agent│  │  │           │
         │  │  │  │   (Claude 3)     │  │  │           │
         │  │  │  └──┬──────────┬────┘  │  │           │
         │  │  │     ↓          ↓       │  │           │
         │  │  │ Analysis   Policy      │  │           │
         │  │  │  Agent     Agent       │  │           │
         │  │  └─────────────────────────┘  │           │
         │  └───────────────────────────────┘           │
         │                                               │
         │  Fluent Bit · Prometheus · Grafana            │
         │  ArgoCD · Jenkins                             │
         └───────────────────────────────────────────────┘
                    │              │              │
         ┌─────────┴──┐  ┌───────┴────┐  ┌─────┴──────┐
         │ RDS        │  │ DynamoDB   │  │   AOSS     │
         │ PostgreSQL │  │ (검색빈도  │  │ OpenSearch │
         │ + PostGIS  │  │  TF Lock)  │  │ (KB + Log) │
         │ (30M+ 레코드)│  └────────────┘  └────────────┘
         └─────────────┘
                                        S3 (데이터 · 로그 · tfstate)

[Serverless Mailing]
  EventBridge (Cron) → λ 크롤링 → λ AI 분석 → λ SES 발송 → 구독자

[ETL Pipeline]
  공공데이터포털 / V-World → S3 → EventBridge → AWS Batch (ECR) → RDS

[CI/CD]
  GitHub → Jenkins → SonarQube → Kaniko → ECR → ArgoCD → EKS
  GitHub → Amplify (Frontend 자동 배포)

[Security]
  IAM (최소 권한 + MFA) · WAF · GuardDuty · CloudTrail · KMS · Secrets Manager
```

---

## 레포지토리 구조

| 레포지토리 | 언어 | 설명 |
|-----------|------|------|
| [`izza-back`](https://github.com/aws-izza/izza-back) | Java (Spring Boot) | 입지 검색 · 비교 분석 API 서버. PostGIS 공간 쿼리, 버킷 단위 집계 |
| [`izza-front`](https://github.com/aws-izza/izza-front) | JavaScript (React) | 프론트엔드. Kakao Map 지도 시각화, 분석 대시보드, 보고서 열람 |
| [`izza-ai`](https://github.com/aws-izza/izza-ai) | Python (FastAPI) | AI 보고서 생성 서버. Strands Agents SDK 멀티 에이전트 아키텍처 |
| [`izza-autocomplete-server`](https://github.com/aws-izza/izza-autocomplete-server) | Go (Gin) | 검색어 자동완성 서버. Trie 자료구조 In-Memory, DynamoDB 인기도 추적 |
| [`izza-email`](https://github.com/aws-izza/izza-email) | Python (Lambda) | 서버리스 메일링. EventBridge → Lambda × 3 → SES 뉴스레터 자동 발송 |
| [`izza-iac`](https://github.com/aws-izza/izza-iac) | HCL (Terraform) | 인프라 코드. VPC, EKS, RDS, S3 등 전체 인프라 IaC |
| [`izza-cd`](https://github.com/aws-izza/izza-cd) | YAML | ArgoCD GitOps 매니페스트. EKS 배포 관리 |
| [`izza-monitoring`](https://github.com/aws-izza/izza-monitoring) | Shell | 모니터링 설정. Fluent Bit, Prometheus, Grafana, Slack 알림 |

---

## 기술 스택

<div align="center">

### Application
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![Gin](https://img.shields.io/badge/Gin_(Go)-00ADD8?style=flat-square&logo=go&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black)

### AWS Services
![EKS](https://img.shields.io/badge/EKS-FF9900?style=flat-square&logo=amazonaws&logoColor=white)
![RDS](https://img.shields.io/badge/RDS_(PostgreSQL)-527FFF?style=flat-square&logo=amazonrds&logoColor=white)
![Bedrock](https://img.shields.io/badge/Bedrock_(Claude_3)-01A88D?style=flat-square&logo=amazonaws&logoColor=white)
![S3](https://img.shields.io/badge/S3-569A31?style=flat-square&logo=amazons3&logoColor=white)
![Lambda](https://img.shields.io/badge/Lambda-FF9900?style=flat-square&logo=awslambda&logoColor=white)
![EventBridge](https://img.shields.io/badge/EventBridge-E7157B?style=flat-square&logo=amazonaws&logoColor=white)
![OpenSearch](https://img.shields.io/badge/OpenSearch-005EB8?style=flat-square&logo=opensearch&logoColor=white)
![DynamoDB](https://img.shields.io/badge/DynamoDB-4053D6?style=flat-square&logo=amazondynamodb&logoColor=white)
![Amplify](https://img.shields.io/badge/Amplify-FF9900?style=flat-square&logo=awsamplify&logoColor=white)
![CloudFront](https://img.shields.io/badge/CloudFront-8C4FFF?style=flat-square&logo=amazonaws&logoColor=white)

### AI
![Strands Agents](https://img.shields.io/badge/Strands_Agents_SDK-7C3AED?style=flat-square)
![Titan Embedding](https://img.shields.io/badge/Titan_Embedding-01A88D?style=flat-square)
![PostGIS](https://img.shields.io/badge/PostGIS-336791?style=flat-square&logo=postgresql&logoColor=white)

### DevOps & Monitoring
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat-square&logo=jenkins&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=flat-square&logo=argo&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square&logo=grafana&logoColor=white)
![Fluent Bit](https://img.shields.io/badge/Fluent_Bit-49BDA5?style=flat-square&logo=fluentbit&logoColor=white)

### Security
![WAF](https://img.shields.io/badge/WAF-DD344C?style=flat-square&logo=amazonaws&logoColor=white)
![GuardDuty](https://img.shields.io/badge/GuardDuty-DD344C?style=flat-square&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/IAM-DD344C?style=flat-square&logo=amazonaws&logoColor=white)
![KMS](https://img.shields.io/badge/KMS-DD344C?style=flat-square&logo=amazonaws&logoColor=white)

### Cooperation
![Jira](https://img.shields.io/badge/Jira-0052CC?style=flat-square&logo=jira&logoColor=white)
![Confluence](https://img.shields.io/badge/Confluence-172B4D?style=flat-square&logo=confluence&logoColor=white)
![Slack](https://img.shields.io/badge/Slack-4A154B?style=flat-square&logo=slack&logoColor=white)

</div>

---

## 팀 구성

| 이름 | 역할 |
|------|------|
| 조진복 | 팀장, 프로젝트 총괄 |
| 김수인 | Frontend (React, Kakao Map), UI/UX |
| 김현교 | 공간DB (PostGIS, ETL), AI (Strands Agents, Bedrock), 로그 파이프라인 |
| 심혜진 | Backend (Spring Boot), 입지 분석 API |
| 이상준 | 인프라 (EKS, Terraform), CI/CD, 보안 |
| 최세민 | Backend (검색 최적화), 모니터링 (Prometheus, Grafana) |

---

## 📄 라이선스

이 프로젝트는 AWS Cloud School 9기 교육 과정의 일환으로 제작되었습니다.

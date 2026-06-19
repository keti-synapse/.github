<div align="center">

# SYNAPSE

**텔레오퍼레이션 로봇 플리트 플랫폼**  
*Teleoperated Robot Fleet Platform*

KETI 지능로보틱스연구센터 · STAR TEAM

---

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white&style=flat-square)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white&style=flat-square)
![WebRTC](https://img.shields.io/badge/WebRTC-333333?logo=webrtc&logoColor=white&style=flat-square)
![aiortc](https://img.shields.io/badge/aiortc-P2P_Transport-8B7CF8?style=flat-square)
![License](https://img.shields.io/badge/License-Internal-3D526A?style=flat-square)

</div>

---

## 개요

SYNAPSE는 **원격 조종 로봇 플리트**를 위한 통합 플랫폼입니다.  
로봇과 오퍼레이터는 **WebRTC P2P**로 직접 연결되고, 서버는 시그널링과 플리트 상태를 중개합니다.  
네트워크 단절 이후에도 링크가 **자동으로 복구**됩니다.

```
[오퍼레이터 브라우저]
        │ WebRTC P2P (data channel + video)
        ▼
   [ nexus ] ─── REST / WebSocket ───► [ nexus_ui ]
        │ WebSocket (signaling)
        ▼
[ tom_and_gerri ]
        │
        └── import ──► [ synapse ] ◄── import ── [ nexus ]
                      (공통 표준 레이어)
```

---

## 레포지토리

<table>
<thead>
<tr>
<th>레포지토리</th>
<th>역할</th>
<th>설명</th>
<th>언어</th>
<th>링크</th>
</tr>
</thead>
<tbody>

<tr>
<td><b>🟣 synapse</b></td>
<td>공통 표준 레이어</td>
<td>토픽 · 메시지 · 열거형 · 모델 · 빌더 정의. 클린 아키텍처 최내층. <code>nexus</code>와 <code>tom_and_gerri</code>가 import.</td>
<td>Python</td>
<td><a href="https://github.com/keti-synapse/synapse">→ 바로가기</a></td>
</tr>

<tr>
<td><b>🟢 nexus</b></td>
<td>시그널링 서버</td>
<td>FastAPI + WebSocket 기반 시그널링/릴레이 게이트웨이. 플리트 상태 캐시 및 REST API. WebRTC offer/answer/ICE 중개.</td>
<td>Python</td>
<td><a href="https://github.com/keti-synapse/nexus">→ 바로가기</a></td>
</tr>

<tr>
<td><b>🔵 nexus_ui</b></td>
<td>관제 대시보드</td>
<td>플리트 상태 모니터링 UI. Alpine.js + Tailwind CSS. 빌드 단계 없는 순수 프론트엔드.</td>
<td>JavaScript / HTML</td>
<td><a href="https://github.com/keti-synapse/nexus_ui">→ 바로가기</a></td>
</tr>

<tr>
<td><b>🔴 tom_and_gerri</b></td>
<td>로봇 제어 코어</td>
<td>단일 로봇 제어 코어. aiortc WebRTC P2P, pypubsub 내부 이벤트 버스, 카메라 추상화(RealSense / webcam). 브라우저 코크핏 UI 포함.</td>
<td>Python</td>
<td><a href="https://github.com/keti-synapse/tom_and_gerri">→ 바로가기</a></td>
</tr>

<tr>
<td><b>⚪ synapse-sdk-python</b></td>
<td>Python SDK <i>(개발 예정)</i></td>
<td>플리트 REST API + WebRTC 풀 라이프사이클 래핑. 오퍼레이터·로봇 역할 모두 지원.</td>
<td>Python</td>
<td><i>Coming soon</i></td>
</tr>

<tr>
<td><b>⚪ synapse-sdk-ts</b></td>
<td>TypeScript SDK <i>(개발 예정)</i></td>
<td>브라우저 네이티브 WebRTC 래핑. cockpit.js / nexus_ui 마이그레이션 타깃.</td>
<td>TypeScript</td>
<td><i>Coming soon</i></td>
</tr>

<tr>
<td><b>⚪ synapse-sdk-unity</b></td>
<td>C# / Unity SDK <i>(개발 예정)</i></td>
<td>XR · 시뮬레이션 환경에서 로봇 제어 및 스트림 수신.</td>
<td>C# / Unity</td>
<td><i>Coming soon</i></td>
</tr>

</tbody>
</table>

---

## 기술 스택

<table>
<tr>
<td><b>Server</b></td>
<td>FastAPI · Uvicorn · Pydantic 2 · websockets · SQLModel · Redis</td>
</tr>
<tr>
<td><b>Transport</b></td>
<td>WebRTC (aiortc) · WebSocket · ICE / STUN / TURN</td>
</tr>
<tr>
<td><b>Robot</b></td>
<td>pypubsub · OpenCV · Intel RealSense · PyAV (aiortc video)</td>
</tr>
<tr>
<td><b>Dashboard</b></td>
<td>Alpine.js 3 · Tailwind CSS 3 · ES2022 · Native WebRTC API</td>
</tr>
<tr>
<td><b>Testing</b></td>
<td>pytest · httpx · GitHub Actions</td>
</tr>
</table>

---

## 설계 원칙

| # | 원칙 | 내용 |
|---|------|------|
| 1 | **차원 분리** | `raw_status` (SDK 원본) · `base_state` (시스템 종합) · `mission_state` (미션) 를 절대 섞지 않는다 |
| 2 | **관용 처리 (Postel's Law)** | 받을 땐 어떤 형태든, 보낼 땐 표준 dict. `ValidationError`로 죽이지 않고 워닝 + 부분 비활성화 |
| 3 | **동적 토픽 금지** | `command.{capability}` 같은 동적 토픽 금지. 토픽 = 카테고리, 세부 = value |
| 4 | **표준 준수 = 호환성** | 표준만 지키면 외부 도구가 추가 개발 없이 모든 로봇과 동작 |

---

<div align="center">
<sub>KETI 지능로보틱스연구센터 STAR TEAM &nbsp;·&nbsp; <a href="https://github.com/keti-synapse">github.com/keti-synapse</a></sub>
</div>

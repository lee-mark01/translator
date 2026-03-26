# 시스템 아키텍처 설계

## 전체 구조

```
[목사님 마이크]
      ↓
[교회 믹서] ── OBS 노이즈 게이트 필터 적용
      ↓
[OBS 컴퓨터] ──→ YouTube (기존 그대로)
      ↓ VB-Audio Virtual Cable (또는 FFmpeg dshow)
[통역 앱 서버 (같은 PC)]
      ↓
  ┌───┴────────────────────┐
  ↓                        ↓
[영어 파이프라인]      [중국어 파이프라인]
  │                        │
  ├ Azure STT (한→텍스트)   ├ Azure STT (공유)
  ├ GPT-4o-mini (→영어)    ├ GPT-4o-mini (→중국어)
  ├ Google TTS (→영어음성)  ├ Google TTS (→중국어음성)
  ↓                        ↓
[WebSocket 중계]
      ↓
[청취자 웹앱 (모바일 브라우저)]
  ├ 음성 재생 (큐 기반 순차 재생)
  └ 자막 표시 (스크롤 가능)
```

## 컴포넌트 설계

### 1. Audio Capture Module
- OBS에서 가상 오디오 케이블(VB-Audio)로 출력
- **캡처 방식 (우선순위)**:
  1. SoX: `sox -t waveaudio "VB-Audio Virtual Cable" -t raw -r 16000 -b 16 -c 1 -`
     → Node.js child_process.spawn으로 stdin PCM 스트림 읽기
  2. FFmpeg (fallback): `ffmpeg -f dshow -i audio="VB-Audio Virtual Cable" ...`
     → OBS에 이미 번들되어 있을 가능성 높음
  3. ~~node-portaudio~~ (비추: Windows 빌드 문제 빈번)
- 포맷: 16kHz, 16-bit, mono PCM
- **PoC 필수 항목**: Windows에서 VB-Audio 디바이스를 안정적으로 잡는지 검증

### 2. Noise Filtering (노이즈 제거)
- **1차: OBS 노이즈 게이트 필터** (추가 비용 $0, 코드 불필요)
  - OBS → 오디오 소스 → 필터 → "Noise Gate" 추가
  - 목사님 마이크 볼륨보다 작은 소리(회중 아멘, 기침) 자동 차단
- **2차: Azure STT 자체 노이즈 내성**
  - 마이크 직접 입력은 회중 소리 대비 음량 차이가 커서 인식률 영향 적음
- **3차 (필요시): 후처리 필터링**
  - STT 결과에서 짧은 단독 "아멘" 문장 감지 시 번역 대상에서 제외
  - 규칙: 2어절 이하 + 감탄사 패턴 → 스킵

### 3. STT Pipeline
- Azure Speech Services (Streaming)
- 한국어 실시간 인식
- 문장 완성 감지 (endpoint detection)
- 출력: 타임스탬프 포함 한국어 텍스트
- **실패 시**: 클라이언트에 "인식 중..." 상태 표시

### 4. Translation Pipeline
- OpenAI GPT-4o-mini
- 시스템 프롬프트: 참예수교회 용어집 + 성경 맥락 지시
- 이전 5-10문장 컨텍스트 유지 (맥락 번역)
- 영어/중국어 병렬 번역 (동시 API 호출)
- 출력: 번역된 텍스트
- **타임아웃**: 5초. 초과 시 해당 문장 스킵, 텍스트에 "[번역 지연]" 표시
- **실패 시**: 한국어 원문을 텍스트로 표시 (graceful degradation)

### 5. TTS Pipeline
- Google Cloud TTS
- 영어: en-US WaveNet 음성
- 중국어: cmn-CN WaveNet 음성
- 스트리밍 오디오 생성
- 출력: 오디오 청크 (MP3/OGG)
- **타임아웃**: 5초. 초과 시 텍스트만 전송
- **실패 시**: 번역 텍스트만 자막으로 표시 (음성 없이 진행)

### 6. TTS Audio Queue (클라이언트 측)
- **큐 기반 순차 재생**: 서버에서 sequence number 부여 → 클라이언트가 순서대로 재생
- **밀림 방지 로직**:
  - 큐에 3개 이상 대기 시 → 중간 항목은 텍스트만 표시, 최신 오디오만 재생
  - 설교 끝났는데 통역이 계속 나오는 상황 방지
- **재생 상태**: 현재 재생 중 / 대기 중 / 스킵됨 추적

### 7. WebSocket Server
- 클라이언트 연결 관리
- 언어별 채널 분리 (en, zh)
- 전송 데이터:
  ```json
  {
    "seq": 42,
    "audio": "<base64 chunk>",
    "text": "translated text",
    "originalText": "한국어 원문",
    "timestamp": 1711440000,
    "status": "ok" | "tts_failed" | "translate_failed"
  }
  ```
- 최근 30초 버퍼 유지 (재연결 복구용)
- 전체 세션 기록 메모리에 유지 (스크롤백용)

### 8. Web Client (청취자 앱)
- 단일 페이지 웹앱 (SPA)
- QR코드로 접속
- 화면 구성:
  - 언어 선택 버튼 (English / 中文)
  - 실시간 자막 영역 (자동 스크롤 + 수동 스크롤)
  - 음성 재생 컨트롤 (볼륨)
  - 연결 상태 표시 (연결됨 / 재연결 중... / 오프라인)
- 자동 재연결 로직 (1초 내 재연결 시도)
- 재연결 시 놓친 텍스트 백필
- TTS 오디오 큐 관리 (섹션 6 참조)

### 9. Storage (P1)
- SQLite (교회 컴퓨터 로컬)
- 테이블: sessions, transcripts
- 설교별 원문 + 번역 + 타임스탬프 저장
- 검증/수정 UI (P1에서 구현)

## API Fallback 전략

각 단계가 독립적으로 실패해도 가능한 범위까지 진행:

```
STT 실패   → "인식 중..." 표시. 할 수 있는 게 없음. 재연결 시도.
번역 실패   → 한국어 원문 텍스트 표시. 음성 없음.
TTS 실패   → 번역 텍스트만 자막 표시. 음성 없음.
WebSocket  → 자동 재연결 + 놓친 텍스트 백필.
```

타임아웃: 모든 API 호출에 5초 제한. 초과 시 해당 문장은 다음 가능한 단계로 전환.

## 데이터 흐름 (시퀀스)

```
1. 마이크 오디오 → OBS 노이즈 게이트 → Virtual Cable → SoX → Node.js 캡처
2. 오디오 청크 → Azure STT (스트리밍)
3. Azure STT → 한국어 문장 완성 이벤트
4. (후처리) 짧은 감탄사 필터링
5. 한국어 텍스트 → GPT-4o-mini (영어/중국어 병렬 번역, 5초 타임아웃)
6. 번역 텍스트 → Google TTS (영어/중국어 병렬 음성 생성, 5초 타임아웃)
7. seq 번호 + 텍스트 + 오디오 + status → WebSocket → 청취자 앱
8. 청취자 앱 → 큐에 추가 → 순차 재생 + 자막 표시
9. (P1) 전체 기록 → SQLite 저장
```

## OBS 컴퓨터 부하 관리

### 사전 측정 (PoC 항목)
- 예배 중 OBS 송출 상태에서 작업 관리자로 CPU/RAM/GPU 측정
- **기준**: OBS만으로 CPU 70% 이상이면 분리 고려

### Node.js 서버 예상 부하
- API 중계 역할만 수행 → CPU 5% 미만, RAM 200-500MB
- 20명 WebSocket 연결 → 미미한 부하

### 부하 초과 시 대안
1. OBS 인코딩을 GPU(NVENC)로 전환 → CPU 여유 확보
2. 별도 미니PC 추가 → Node.js 서버 분리 (VB-Audio 출력을 네트워크 전송)

## API 비용 구조 (월 12시간, 2언어 기준)

| 서비스 | 단가 | 월 비용 | 비고 |
|--------|------|---------|------|
| Azure STT | $1.02/hr | ~$12 | 5hr/월 무료 → 실질 ~$7 |
| GPT-4o-mini | ~$0.15/hr x 2lang | ~$4 | 토큰 기반 종량제 |
| Google TTS | 무료 (4M자/월) | $0 | 무료 티어 내 |
| **합계** | | **~$11-17** | |

## 품질 전환 옵션

| 모드 | 번역 엔진 | 월 비용 | 용도 |
|------|----------|---------|------|
| 절약 | GPT-4o-mini | ~$17 | 일반 예배 |
| 고품질 | GPT-4o | ~$53 | 특별 집회/중요 설교 |

관리자 설정에서 전환 가능하도록 구현.

## 네트워크 요구사항

| 항목 | 필요량 |
|------|--------|
| 업로드 (서버→청취자) | ~5Mbps (20명 기준) |
| 다운로드 (API 호출) | ~2Mbps |
| 청취자 1명당 | ~200kbps |

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| 서버 런타임 | Node.js |
| 웹 프레임워크 | Express |
| WebSocket | Socket.io |
| 오디오 캡처 | SoX (child_process) / FFmpeg (fallback) |
| STT | Azure Speech SDK (@azure/cognitiveservices-speech-sdk) |
| 번역 | OpenAI API (openai npm) |
| TTS | Google Cloud TTS (@google-cloud/text-to-speech) |
| 프론트엔드 | Vanilla JS 또는 React (경량) |
| DB (P1) | SQLite (better-sqlite3) |
| 배포 | Docker (OBS 컴퓨터에서 실행) |

## PoC (개념 증명) — 태스크 분해 전 필수 검증

### PoC-1: 오디오 캡처 검증
- Windows에서 VB-Audio → SoX → Node.js PCM 스트림이 안정적으로 동작하는지
- 실패 시 FFmpeg 방식으로 전환
- **이게 안 되면 전체 파이프라인이 막힘 → 최우선 검증**

### PoC-2: End-to-End 지연 측정
- 실제 설교 녹음 샘플로 Azure STT → GPT 번역 → Google TTS 전체 지연 측정
- **목표: 3초 이내**
- 실패 시 병렬화 최적화 또는 API 대안 검토

### PoC-3: OBS 컴퓨터 부하 측정
- OBS YouTube 송출 중 CPU/RAM/GPU 사용률 측정
- Node.js 서버 추가 시 OBS 프레임 드랍 발생 여부 확인

# Church Translator App

## Project
참예수교회(True Jesus Church) 실시간 설교 통역 앱. 한국어 → 영어/중국어.

## Rules
- 모호하면 반드시 질문. 자의적 해석 금지.
- 사용자에게 직접 확인이 필요한 사항(교회 현장 환경, PC 스펙, 네트워크 등)은 명시적으로 확인 요청.
- 한 번에 하나의 TASK만 구현.
- 세션 시작: docs/status.md → docs/history/ 확인 → 새 히스토리 생성.
- 세션 종료: docs/status.md 업데이트 → 히스토리 저장 → tasks 정리.
- 버그 해결 과정은 docs/troubleshooting/에 기록.

## Docs Structure
- `docs/requires/` — 요구사항
- `docs/spec/` — 설계
- `docs/tasks/` — 진행중 태스크
- `docs/complete/` — 완료된 태스크
- `docs/history/` — 세션 히스토리
- `docs/troubleshooting/` — 문제해결 기록
- `docs/status.md` — 현재 프로젝트 상태 (외부 기억장치)

## Tech Stack
- BE: Node.js (OBS 컴퓨터에서 실행)
- FE: 웹앱 (QR 접속, 설치 불필요)
- STT: Azure Speech Services (한국어)
- Translation: OpenAI GPT-4o-mini (성경/신학 용어집 포함)
- TTS: Google Cloud TTS
- Audio Input: OBS 가상 오디오 출력 (VB-Audio Virtual Cable)

# Project Status

## Current Phase
설계 단계 (Design Phase)

## Last Session
- Date: 2026-03-26
- Session: 002
- Summary: 설계 리뷰 피드백 반영 (TTS 큐, fallback, 노이즈, 부하, 오디오 캡처)

## Active Task
대기 중 — 사용자가 교회 현장 정보 확인 후 PoC 진행 예정

## Pending User Confirmation
1. OBS 컴퓨터 스펙 (CPU, RAM, GPU, Windows 버전)
2. OBS 송출 중 CPU/RAM/GPU 사용률
3. 교회 인터넷 속도 (speed test)
4. 교회 음향 시스템 (믹서 모델, OBS 오디오 입력 방식)

## Decisions Made
1. 오디오 입력: OBS 컴퓨터에서 가상 오디오 케이블로 연결
2. 클라이언트: 웹앱 (QR 접속, 앱 설치 불필요)
3. 호스팅: 교회 OBS 컴퓨터 (클라우드 서버 불필요)
4. API: Azure STT + GPT-4o-mini + Google TTS (월 $11-17)
5. 언어: 영어, 중국어 (추후 확장 가능)
6. 우선순위: 정확한 통역 > 속도
7. 오디오 캡처: SoX 우선, FFmpeg fallback
8. TTS 큐: 순차 재생, 3개 밀리면 스킵
9. API 타임아웃: 전부 5초, 단계별 graceful degradation
10. 노이즈: OBS 노이즈 게이트 → Azure 내성 → 후처리 필터
11. PoC 3개 우선 검증 후 태스크 분해

## Blockers
없음 — PoC 검증 준비 완료

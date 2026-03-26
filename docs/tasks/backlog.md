# Task Backlog

## Phase 0: 설계 및 PoC
- [x] 요구사항 정의 (docs/requires/requirements.md)
- [x] 아키텍처 설계 (docs/spec/architecture.md)
- [x] 용어집 초안 (docs/spec/glossary.md)
- [x] 설계 리뷰 피드백 반영 (TTS 큐, fallback, 노이즈, 부하)
- [ ] **PoC-1**: Windows VB-Audio → SoX → Node.js 오디오 캡처 검증 (최우선)
- [ ] **PoC-2**: Azure STT → GPT 번역 → Google TTS end-to-end 지연 측정 (목표 3초)
- [ ] **PoC-3**: OBS 컴퓨터 부하 측정 (OBS 송출 중 CPU/RAM/GPU)
- [ ] PoC 결과에 따른 태스크 분해

## Phase 1: MVP (실시간 통역)
> PoC 검증 후 세부 태스크 분해 예정

- [ ] TASK-001: 프로젝트 초기화 (Node.js, 패키지 구조)
- [ ] TASK-002: 오디오 캡처 모듈 (SoX/FFmpeg → Node.js)
- [ ] TASK-003: Azure STT 연동 (한국어 실시간 인식)
- [ ] TASK-004: GPT-4o-mini 번역 파이프라인 (용어집 + 컨텍스트 + 5초 타임아웃)
- [ ] TASK-005: Google TTS 연동 (영어/중국어 음성 + 5초 타임아웃)
- [ ] TASK-006: API fallback 로직 (단계별 graceful degradation)
- [ ] TASK-007: WebSocket 서버 (seq 번호, 언어별 채널, 버퍼)
- [ ] TASK-008: 웹 클라이언트 (QR 접속, 자막 + 음성 큐 재생)
- [ ] TASK-009: TTS 오디오 큐 관리 (순차 재생 + 밀림 방지 스킵)
- [ ] TASK-010: 재연결 + 버퍼 복구 로직
- [ ] TASK-011: 노이즈 후처리 (짧은 감탄사 필터링)
- [ ] TASK-012: 통합 테스트 (전체 파이프라인)
- [ ] TASK-013: OBS 컴퓨터 환경 배포 테스트

## Phase 2: 기록 & 검증
- [ ] TASK-014: SQLite 저장 (설교 기록)
- [ ] TASK-015: 번역 검증/수정 UI
- [ ] TASK-016: 설교 기록 조회 UI

## Phase 3: 향후
- [ ] 추가 언어 지원
- [ ] 멀티테넌트 (다른 교회)

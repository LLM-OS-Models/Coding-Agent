# Experiment Card: Coding-Agent

## task_type
`coding_agent`

## 목적
문서 이해, 파일 탐색, 패치 생성, 터미널 실행, 테스트 통과까지 포함하는 repo-grounded coding agent를 구축한다.

## 핵심 지표
- file_selection_recall — 정답 파일 대비 선택 재현율 (0~1)
- patch_present — diff/patch 블록 존재 여부 (0/1)
- test_plan_recall — 정답 테스트 대비 재현율 (0~1)

## 평가 실행
```bash
bash eval/run_phase1.sh
bash eval/run_phase2.sh
```

## 평가 모델
- Phase 1: 8개 모델
- Phase 2: Qwen3.6-27B + LFM 모델

# Labeling Guide: Coding-Agent

## Goal

- repo-grounded planning trajectory를 학습 가능한 구조로 고정한다.
- 단순 정답 문장 대신 file selection, patch plan, tests를 분리 저장한다.

## Required Fields

- `repo_snapshot_path`
- `repo_tree_summary`
- `selected_files`
- `patch_plan`
- `target_tests`
- `expected_diff_summary`

## Labeling Rules

- `selected_files`는 실제 수정 후보 파일만 넣는다.
- `patch_plan`은 "적절한 수정" 같은 placeholder를 금지한다.
- 테스트 계획은 수정 파일과 연결되어야 한다.
- plan-only task와 patch+test task를 난이도 태그로 분리한다.

## Verification

1. repo snapshot 열림 확인
2. selected files 존재 확인
3. patch plan 구체성 확인
4. target tests 실행 가능성 확인
5. difficulty / task type 태그 확인

## Common Mistakes

- repo tree summary 없이 patch plan만 저장
- selected files가 너무 넓어 supervision이 약해짐
- target tests가 generic해서 patch plan과 연결되지 않음

# Finetuning Plan: Coding-Agent

## Current State

- 현재 저장된 결과는 `v0` 4샘플 반복 실행 중심이다.
- `v1` 평가용 `repo_snapshot` 자산이 30/30 누락이다.
- 현재 SFT 데이터 30/30이 `PATCH_PLAN: 적절한 수정을 수행합니다.` 형태라 학습 신호가 약하다.

## Priority

- 우선순위: 낮음
- 이유: 모델 미세조정보다 repo tree, patch trace, test outcome이 포함된 실제 trajectory 데이터가 더 중요하다.

## Base Models

- Teacher: `Qwen/Qwen3-Coder-30B-A3B-Instruct`
- Secondary teacher: `Qwen/Qwen3-Coder-480B-A35B-Instruct`
- Student: `Qwen/Qwen3-8B`
- Small student: `Qwen/Qwen3-4B`

## Phase 0

1. `repos/*.tar.gz` 30개를 먼저 복구한다.
2. placeholder `PATCH_PLAN` 데이터를 폐기한다.
3. 단순 정답 파일 목록이 아니라 다음 trajectory를 새로 만든다.
   - repo tree summary
   - selected files
   - concrete patch plan
   - candidate diff
   - target tests
   - regression tests
4. hard 샘플은 file-selection only / plan-only / patch+test로 분해해서 난이도 계층을 만든다.

## Phase 1

- 목표: teacher를 `repo-grounded planning`에 맞게 SFT
- 방법: LoRA/QLoRA 먼저
- 권장 시작점
  - `max_seq_length=8192`
  - `per_device_train_batch_size=1`
  - `gradient_accumulation_steps=8`
  - `learning_rate=1e-4`
  - `lora_r=32`
  - `num_train_epochs=1`

## Phase 2

- teacher 출력을 이용해 student distillation
- 입력은 issue text만 두지 말고 repo tree 요약도 함께 넣는다.
- distillation 타깃은 최종 문자열만이 아니라 다음 순서로 맞춘다.
  - `FILES`
  - `PATCH_PLAN`
  - `TESTS`

## Phase 3

- 실제 patch applicability와 target test pass를 reward로 쓰는 RL 또는 reranking을 붙인다.
- 이 단계는 repo snapshot과 sandbox 실행이 안정화된 후에만 진행한다.

## Model Notes

- `Qwen3-Coder-30B-A3B-Instruct`는 non-thinking only이며 긴 컨텍스트와 tool calling에 강하다.
- coding agent teacher는 일반 `Qwen3`보다 coder 계열이 더 자연스럽다.

## Exit Criteria

- `v1`에서 `file_selection_recall >= 0.75`
- `patch_present >= 0.95`
- `test_plan_recall >= 0.85`
- hard split에서도 easy/medium 대비 급락하지 않을 것

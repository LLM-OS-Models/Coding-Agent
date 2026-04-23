# Pilot Run Sheet: Coding-Agent

## Objective

- repo-grounded planning이 student에 증류 가능한지 본다.
- 단순 답변형 SFT가 아니라 file selection과 test planning까지 학습 단위로 고정한다.

## Run IDs

- `CODE-P0`: repo snapshot + trajectory schema 확정
- `CODE-P1`: teacher trajectory SFT design freeze
- `CODE-P2`: `Qwen3-8B` student distillation pilot
- `CODE-P3`: `Qwen3-4B` small student feasibility

## Dataset Gate

- repo snapshot missing `0`
- placeholder patch plan `0`
- trajectory fields 완료:
  - repo tree summary
  - selected files
  - patch plan
  - target tests
  - regression tests

## Model Matrix

| Run ID | Model | Context | Rank | Target | Purpose |
|---|---|---:|---:|---|---|
| CODE-P1 | `Qwen3-Coder-30B-A3B` | 8192 | 32 | teacher SFT | teacher formatting lock |
| CODE-P2 | `Qwen3-8B` | 8192 | 32 | student distill | primary student |
| CODE-P3 | `Qwen3-4B` | 8192 | 32 | student distill | small-model floor |

## Fixed Decisions

- output contract: `FILES`, `PATCH_PLAN`, `TESTS`
- repo tree summary는 입력에 포함
- first stage: plan-level SFT only
- patch applicability reward는 후행 단계로 미룸

## Primary Metrics

- `file_selection_recall`
- `patch_present`
- `test_plan_recall`

## Slice Metrics

- plan-only split
- patch+test split
- small repo split
- multi-file repo split

## Accept

- `file_selection_recall >= 0.75`
- `test_plan_recall >= 0.85`
- hard split에서 format collapse 없음

## Reject

- selected files가 흔들려 patch plan이 무의미해짐
- repo context를 넣어도 improvement가 거의 없음
- 4B와 8B 모두 teacher structure를 재현하지 못함

## Review Questions

1. 모델이 patch보다 file selection에서 먼저 무너지는가
2. repo tree summary 길이가 성능에 크게 영향을 주는가
3. student distillation 전에 teacher formatting부터 다시 고정해야 하는가

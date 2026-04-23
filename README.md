# Coding-Agent

Repo-grounded 코딩 에이전트 평가 트랙.

이슈 설명과 작업 공간 경로가 주어지면, 모델이 수정할 파일을 식별하고, 패치 계획을 수립하며, 테스트 계획을 생성하는 능력을 평가한다. 실제 코드 실행은 하지 않고 구조적 응답의 정확도를 측정한다.

## 평가 메트릭

| 메트릭 | 설명 |
|--------|------|
| `file_selection_recall` | gold target_files 중 모델이 올바르게 식별한 비율 |
| `patch_present` | 모델 출력에 diff/patch 블록이 포함되어 있는지 (0/1) |
| `test_plan_recall` | gold target_tests 중 모델이 올바르게 제안한 비율 (fuzzy matching) |

Fuzzy matching normalizes path separators and compares word overlap (threshold=0.6)

**성공 조건**: `file_selection_recall > 0 AND test_plan_recall > 0 AND patch_present > 0`

## 샘플 데이터 형식

```json
{
  "sample_id": "code_0001",
  "user_query": "로그인 실패 시 에러 메시지가 비어 있는 문제를 수정하라.",
  "artifacts": {
    "repo_snapshot": "repos/auth_service_v3.tar.gz",
    "issue_text": "When login fails with invalid token, UI receives empty message.",
    "workspace_path": "/workspace/auth_service"
  },
  "gold": {
    "target_files": ["src/auth/login.py", "src/auth/handlers.py"],
    "target_tests": ["tests/test_login.py::test_invalid_token_message"],
    "regression_tests": ["tests/test_login.py", "tests/test_auth_flow.py"]
  }
}
```

## 모델 출력 형식

```
FILES: [path/to/file.py, path/to/other.py]
PATCH_PLAN: 변경 사항 설명
TESTS: [tests/test_something.py::test_case]
```

## 프로젝트 구조

```
Coding-Agent/
├── README.md
├── pyproject.toml
├── eval/
│   ├── internal/
│   │   └── v0.jsonl        # 평가 데이터셋 (2샘플)
│   └── results/
├── tests/
└── data/
```

## 실행

```bash
uv sync

llm-os-eval run coding_agent \
  --model Qwen/Qwen3-4B \
  --samples eval/internal/v0.jsonl \
  --output eval/results/Qwen3-4B_v0.jsonl \
  --base-url http://localhost:8001/v1
```

## 벤치마크 결과 (2026-04-23, Round 3)

| 모델 | Size | file_recall | patch_present | test_recall | 성공률 |
|------|------|-------------|---------------|-------------|--------|
| Qwen3-0.6B | 0.6B | 0% | **50%** | 0% | 0% |
| 나머지 전체 | — | 0% | 0% | 0% | 0% |

Coding Agent는 모든 모델에게 가장 어려운 트랙이다. 모델이 실제 repo 구조를 알 수 없으므로 정확한 파일 경로를 예측하기 거의 불가능하다. 이 트랙은 실제 파일 시스템 접근이 가능한 에이전트(도구 사용 포함)를 전제로 설계되었다.

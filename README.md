# Codex Skills

개인용 Codex 스킬 모음입니다. 각 디렉터리는 독립적으로 설치하고 호출할 수 있는 하나의 스킬을 담습니다.

## 포함된 스킬

| 스킬 | 용도 |
| --- | --- |
| [`guideline`](./guideline/README.md) | 낯선 설정·배포·마이그레이션 작업을 전체 흐름부터 설명하고, 확인 가능한 한 단계씩 안내합니다. |
| [`runner`](./runner/README.md) | 요구사항 계약, 승인 게이트, 점진적 구현, 검증과 리뷰를 포함하는 소프트웨어 전달 워크플로를 실행합니다. |

## 디렉터리 구조

```text
.
├── guideline/
│   ├── README.md
│   ├── SKILL.md
│   └── agents/openai.yaml
└── runner/
    ├── README.md
    ├── SKILL.md
    └── agents/openai.yaml
```

- `SKILL.md`: 스킬의 메타데이터와 실행 지침
- `agents/openai.yaml`: Codex UI에 표시할 이름, 설명, 기본 프롬프트
- `README.md`: 사용자를 위한 스킬 소개와 사용법

## 설치

사용할 스킬 디렉터리를 Codex의 스킬 경로에 복사하거나 심볼릭 링크로 연결합니다.

```bash
mkdir -p "$CODEX_HOME/skills"
cp -R guideline "$CODEX_HOME/skills/guideline"
cp -R runner "$CODEX_HOME/skills/runner"
```

`CODEX_HOME`을 별도로 설정하지 않았다면 일반적으로 `~/.codex`를 사용합니다. 설치 후 새 Codex 세션에서 스킬을 호출하세요.

## 사용 예시

```text
$guideline으로 처음 하는 배포 과정을 한 단계씩 안내해 줘.
```

```text
$runner로 이 기능을 요구사항 계약부터 검토·구현해 줘.
```

`runner`는 구현 전에 요구사항 계약과 구현 계획에 대해 각각 명시적인 승인을 요청합니다. 승인 이후에는 승인된 범위 안에서 구현, 검증, 리뷰, 커밋을 자동으로 진행합니다.

## 스킬 추가 및 수정

새 스킬은 다음 최소 구조를 따릅니다.

```text
<skill-name>/
├── SKILL.md
└── agents/openai.yaml
```

변경 시 다음을 확인합니다.

1. `SKILL.md`의 YAML frontmatter에 `name`과 `description`이 있는지 확인합니다.
2. 디렉터리 이름과 frontmatter의 `name`을 일치시킵니다.
3. 설명에는 스킬을 사용해야 할 상황과 사용하지 않아야 할 상황을 명확히 적습니다.
4. `agents/openai.yaml`의 기본 프롬프트에서 `$<skill-name>`을 정확히 참조합니다.
5. 문서의 규칙이 서로 충돌하거나 무한 반복을 만들지 않는지 검토합니다.

각 스킬의 자세한 동작과 사용 예시는 해당 디렉터리의 `README.md`를 참고하세요. 에이전트가 실제로 따르는 동작 계약은 `SKILL.md`가 기준입니다.

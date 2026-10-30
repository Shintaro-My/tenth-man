# tenth-man

[Codex](https://developers.openai.com/codex)와 [Claude Code](https://claude.com/claude-code)에서 사용하는 다중 모델 심의 플러그인(스킬과 참석자 에이전트)입니다. 설계 선택, 정책 검토처럼 한 모델의 답에만 맡겨서는 안 되는 결정에 사용합니다. 서로 다른 모델을 사용하는 위원들이 투표하고 고정된 한 자리가 항상 반대 논거를 맡으므로, 회의가 같은 맹점을 공유한 채 합의할 가능성이 줄어듭니다.

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

투표 위원은 모든 안건을 평가합니다. 별도의 **열 번째 사람**(tenth man)은 자신의 견해와 관계없이 모든 안건에 반대 논거를 펴야 하며, 각 반론을 반박할 수 있는 증거가 무엇인지도 밝혀야 합니다. 메인 세션이 의장을 맡아 세 라운드를 진행하고, 강한 우려를 해소하며, 무엇이 왜 합의되었고 무엇이 미해결로 남았는지 보여 주는 회의록을 작성합니다.

스킬은 회의 계획을 먼저 제시하고 사용자의 승인을 기다립니다. 기다리지 않으려면 질문 앞에 `fast`를 붙입니다(예: `/tenth-man:council fast 우리가 ...해야 할까요?`). 이 경우 의장은 계획을 제시한 뒤 바로 시작하며, 기본값을 직접 선택하고 그 선택 내용을 회의록에 적습니다. 회의를 실행해도 그 결론이 프로젝트에 자동으로 적용되지는 않습니다.

## 진행 방식

1. 의장이 안건, 모델, 규칙, 비용, 출력 폴더를 제안합니다.
2. 1라운드: 위원이 투표하고, 열 번째 사람이 모든 안건에 이의를 제기합니다.
3. 2라운드: 위원이 강한 우려를 재검토하고 가장 중요한 반론에 답합니다.
4. 3라운드: 위원이 회의록 초안에 투표하고, 열 번째 사람이 결과를 증거와 대조해 확인합니다.
5. 의장이 `.tenth-man/<date>-<slug>/minutes.md`를 작성합니다.

투표 선택지는 **찬성**, **조건부 찬성**, **반대**, **강한 우려**입니다. 합의하려면 투표 위원의 과반수가 필요하고 강한 우려가 남아 있지 않아야 합니다. 열 번째 사람에게는 투표권이 없습니다.

## Codex에 설치

### 플러그인으로 설치

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

로컬 체크아웃을 사용하려면 대신 그 경로를 추가합니다.

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

플러그인 스킬은 `$tenth-man:council` 뒤에 질문을 입력하거나 `/skills`에서 선택해 호출합니다.

```text
$tenth-man:council 세션 상태를 Redis에서 서명된 쿠키로 옮겨야 할까요?
```

### 독립 스킬로 설치

`skills/council`을 Codex 스킬 디렉터리에 복사합니다.

```sh
cp -r skills/council ~/.agents/skills/
```

이렇게 설치한 경우 `$council`로 호출합니다.

한 저장소에서만 사용하려면 대신 `<repo>/.agents/skills/`에 복사합니다. 이 저장소의 `.codex/agents/` 아래에는 역할이 정해지지 않은 Codex용 참석자 에이전트 프로필도 들어 있으며, 이 저장소를 체크아웃한 디렉터리에서는 Codex가 이 프로필을 자동으로 찾습니다. 모든 저장소에서 이 프로필을 사용하려면 다음과 같이 복사합니다.

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

이 복사를 하지 않아도 스킬은 동작합니다. 같은 모델을 명시적으로 지정해 서브에이전트를 시작하고, 라운드가 바뀌어도 그 스레드를 이어 갈 수 있기 때문입니다. 플러그인 패키징은 커스텀 에이전트 TOML 파일을 자동으로 설치하지 않습니다.

## Claude Code에 설치

### 플러그인으로 설치

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

로컬 체크아웃을 테스트하려면 다음과 같이 실행합니다.

```sh
claude --plugin-dir /path/to/tenth-man
```

다음과 같이 호출합니다.

```text
/tenth-man:council 세션 상태를 Redis에서 서명된 쿠키로 옮겨야 할까요?
```

### 수동으로 설치

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

수동으로 설치한 경우 명령은 `/council`이며, 참석자 에이전트 이름에는 `tenth-man:` 접두사가 붙지 않습니다.

## 모델 프리셋

의장은 작은 질문에는 `light`를, 그 밖에는 `standard`를 제안합니다. 계획을 승인할 때 프리셋을 고르거나 원하는 참석자를 바꿀 수 있습니다.

열 번째 사람은 위원 중 추론 능력이 가장 강한 모델과 같은 모델로 실행됩니다. 이 순위는 버전 번호가 아니라 측정 결과에 따른 것이며, 아래 표의 열 번째 사람 열은 2026년 10월 기준 순위를 반영합니다. 새 모델의 순위를 알 수 없으면 의장이 사용자에게 묻습니다. 다른 모델을 사용하려면 진행을 승인할 때 말해 주세요.

### Codex

| 프리셋 | 투표 위원 | 열 번째 사람 | 라운드당 실행 수 |
|---|---|---|---|
| `light` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6.1 Sol | 4 |

이 기본값은 현재 Codex 호스트와 계정에서 제공하는 모델만 사용합니다. 의장은 사용할 수 없는 모델을 보고하며, 알리지 않고 다른 모델로 대체하는 일은 없습니다. 호스트의 동시 실행 한도가 회의 규모보다 작으면 참석자 에이전트는 같은 브리핑을 변경 없이 받아 몇 차례에 나누어 실행되며, 모든 참석자 에이전트가 끝날 때까지 집계하지 않습니다.

이에 대응하는 Codex 커스텀 에이전트는 `seat_astra`, `seat_sol`, `seat_terra`, `seat_previous_sol`과 선택 사항인 저비용 `seat_luna`입니다. 각 TOML 정의는 모델과 추론 강도를 고정하고 회의 작업을 읽기 전용으로 유지합니다.

### Claude Code

| 프리셋 | 투표 위원 | 열 번째 사람 | 라운드당 실행 수 |
|---|---|---|---|
| `light` | Fable, Opus, Sonnet | Fable | 4 |
| `standard` | Fable, Opus, Sonnet, Opus 5, Opus 4.8 | Fable | 6 |
| `no-fable` | Opus, Sonnet, Opus 5 | Opus | 4 |

Fable의 순위가 더 높더라도 모든 프리셋에 Opus 참석자를 둡니다. Opus가 Fable이 놓친 문제를 찾을 때가 있기 때문입니다.

Claude 플러그인에는 역할이 정해지지 않은 참석자 에이전트 다섯 개가 `agents/`에 들어 있습니다. 각 실행이 위원 역할을 맡을지 열 번째 사람 역할을 맡을지는 브리핑에서 지정합니다.

### Claude 모델 추가

이전 Claude 모델은 에이전트 정의의 `model:` 필드로 선택할 수 있습니다. 참석자 에이전트 파일을 복사해 `name`, `description`, `model`을 바꾼 다음, 모든 프로젝트에서 사용하려면 `~/.claude/agents/`에, 한 프로젝트에서만 사용하려면 `.claude/agents/`에 저장합니다.

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

본문은 기본 제공 참석자 에이전트와 같이 역할이 정해지지 않은 내용으로 유지합니다. 그러면 의장이 `seat-opus-4-7`을 위원이나 열 번째 사람으로 지정할 수 있습니다.

### 직접 만든 Claude 에이전트 사용

기존 Claude 에이전트 정의라면 어느 것이든 참석자로 쓸 수 있습니다. 의장은 프롬프트 첫머리에서 이번 작업에서는 평소 업무 대신 회의 참석자로 참여한다고 밝힌 뒤, 역할과 공통 브리핑을 전달합니다. 이전 모델에 고정된 리뷰 에이전트가 있을 때 유용합니다.

### 고정한 Claude 모델의 제공이 종료되면

고정한 모델이 더 이상 제공되지 않으면 그 참석자 에이전트는 시작되지 않습니다. 의장은 그 참석자를 빼고 사용자에게 알리며, 알리지 않고 다른 모델로 대체하는 일은 없습니다. 계정에서 사용할 수 있는 모델에 맞게 참석자 에이전트 파일을 삭제하거나 수정하세요.

## 비용과 요구 사항

참석자 한 명당 라운드마다 서브에이전트가 한 번씩 전체 실행됩니다. 세 라운드 동안 Codex `standard` 프리셋은 15회, Claude Code `standard` 프리셋은 18회 실행합니다.

- Codex: 서브에이전트가 활성화된 현재 릴리스와 선택한 모델에 대한 접근 권한.
- Claude Code: Agent 및 SendMessage 도구, 백그라운드 에이전트 완료 알림, 선택한 모델에 대한 접근 권한.

ChatGPT 채팅은 참석자마다 다른 모델로 서브에이전트를 실행할 수 없으므로 현재 지원 대상이 아닙니다.

## 왜 “열 번째 사람”인가?

이 스킬은 2013년 영화 *월드 워 Z*에 나오는 이스라엘의 가상 규칙 “열 번째 사람”에서 착안했습니다. 아홉 사람이 같은 정보로 같은 결론에 이르면, 열 번째 사람은 그들이 틀렸다고 가정하고 그 반대를 조사합니다. 이 규칙은 허구이지만 비슷한 관행은 실제로 있습니다.

- 이스라엘 군 정보부는 1973년 욤 키푸르 전쟁 전에 경고 신호를 잘못 읽은 뒤, 자신의 평가에 이의를 제기하는 기능을 만들었습니다. 이 기능은 흔히 *ipcha mistabra*라고 불리며, 아람어로 “반대가 더 그럴듯하다”는 뜻입니다.
- 탈무드(산헤드린 17a)에는 사형 사건에서 모든 재판관이 유죄에 투표하면 피고인을 석방한다고 기록되어 있습니다. 만장일치라는 결과는 아무도 피고 측 변론을 제대로 펴지 않았음을 시사하기 때문입니다.
- 가톨릭 시성 심사에서는 “악마의 변호인”이 후보자에 반대하는 논거를 펴고, 후보자의 생애와 기적에 관한 주장에서 약점을 검증했습니다. 이 직책은 1587년에 공식화되었고 1983년에 크게 축소되었습니다.

## 라이선스

MIT. [LICENSE](LICENSE)를 참고하세요.

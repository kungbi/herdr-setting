# Herdr 공통 설정

여러 컴퓨터에서 같은 Herdr 테마·단축키와 Agent Picker를 사용하기 위한 저장소입니다.
**설정의 사본과 소스만 공유하며, 실행 중인 작업공간이나 AI 대화를 동기화하지 않습니다.**

## 공유 범위

- `herdr/config.toml`: Dracula 테마, UI 설정, 단축키입니다.
- `herdr/plugins.txt`: 외부 플러그인의 설치 저장소와 현재 설치 ref입니다.
- `plugins/agent-picker/`: 로컬 Agent Picker 소스·테스트·MIT 라이선스입니다.
- `.gitignore`: 검토한 파일만 추적하도록 제한하는 허용 목록입니다.

인증정보, 환경 파일, 회사 프로젝트 경로, 서버 주소, 세션, 로그, 캐시,
완료 시각 기록, 설치된 외부 플러그인의 바이너리는 공유하지 않습니다.
Neovim 개인 설정과 Tailscale 설정도 포함하지 않습니다.

## 대상 환경

- 이 설정은 **Herdr 0.9.1** 환경에서 가져왔습니다. Herdr를 자동으로 업데이트하지 않습니다.
- Agent Picker는 Python 3.11 이상, fzf 0.73.1 이상, macOS 또는 Linux가 필요합니다.
- 외부 플러그인은 각자의 빌드·실행 의존성이 필요합니다. herdr-nvim은 기존 Neovim 설정을 사용합니다.
- reviewr는 Herdr 0.9.1과 맞추기 위해 `v0.45.0`을 유지합니다.

## 다른 컴퓨터에서 처음 적용하기

### 1. 저장소 가져오기

```sh
git clone https://github.com/kungbi/herdr-setting.git "$HOME/herdr-setting"
cd "$HOME/herdr-setting"
herdr --version
sh plugins/agent-picker/run check
```

Herdr 또는 Python·fzf가 없다면 먼저 해당 컴퓨터에 설치합니다.
`run check`는 의존성 확인만 수행하며 Herdr 작업공간에 접속하지 않습니다.

### 2. 외부 플러그인과 Agent Picker 설치하기

아래 명령은 저장된 ref로 설치합니다. 웹 UI인 Ovis는 기본 설치에서 제외합니다.
기존에 다른 버전이 설치돼 있다면 먼저 `herdr plugin list`로 확인합니다.
플러그인 설치 과정에서는 각 프로젝트의 설치·빌드 훅이 실행될 수 있습니다.

```sh
(
    set -eu
    while read -r repository ref; do
        case "$repository" in
            ''|\#*|devswha/herdr-web-ui) continue ;;
        esac
        herdr plugin install "$repository" --ref "$ref" --yes
    done < herdr/plugins.txt
    herdr plugin link "$PWD/plugins/agent-picker" --enabled
)
```

Agent Picker는 이 저장소 안의 디렉터리를 참조합니다. 설치 후 저장소를 이동하거나
삭제하면 링크가 끊어질 수 있습니다. 이 명령은 기존 Neovim 설정을 교체하지 않습니다.

### 3. 설정을 검사한 뒤 백업하고 적용하기

아래 명령은 **현재 컴퓨터의 Herdr 설정을 교체**합니다.
기존 개인 설정을 유지하려면 파일을 먼저 비교하고 필요한 항목만 병합합니다.

```sh
(
    set -eu
    HERDR_CONFIG_PATH="$PWD/herdr/config.toml" herdr config check
    target="${HERDR_CONFIG_PATH:-$HOME/.config/herdr/config.toml}"
    mkdir -p "$(dirname "$target")"
    if [ -e "$target" ]; then
        backup=$(mktemp "${target}.backup.XXXXXX")
        cp -p "$target" "$backup"
        printf '기존 설정 백업: %s\n' "$backup"
    fi
    cp herdr/config.toml "$target"
    herdr config check
)
```

Herdr가 실행 중이라면 **그 Herdr 안의 터미널에서** 다음 명령으로 반영합니다.

```sh
herdr server reload-config
```

Herdr가 실행 중이 아니라면 다음 시작 시 설정을 읽습니다. 재시작이나 업그레이드는 필요하지 않습니다.

## 주요 단축키

기본 prefix는 `Ctrl+B`입니다. prefix를 누르고 다음 키를 누릅니다.

- `g`: Agent Picker입니다.
- `Ctrl+R`: reviewr 코드 리뷰입니다. 기본 `r` resize는 유지합니다.
- `Ctrl+P`: Plugin Manager입니다. 기본 `p` 이전 탭은 유지합니다.
- `a`: Annotate 터미널 캡처입니다.
- `m`: Annotate 관리입니다.
- `o` / `Shift+O` / `Ctrl+O`: 문서 / AI 응답 / 최신 AI 응답 리뷰입니다.
- `Ctrl+E` / `Ctrl+F`: Neovim 사이드바 / 파일 찾기입니다.
- `w` 탐색 모드: `j`/`k`로 workspace를, `↓`/`↑`로 세로 pane을, `h`/`l`로 가로 pane을 이동합니다.

## Ovis 웹 UI는 컴퓨터별로 별도 설정합니다

`herdr/plugins.txt`에는 현재 Ovis ref도 기록돼 있지만 기본 설치 루프는 건너뜁니다.
필요한 컴퓨터에서만 해당 ref로 설치하고, 시작 전에 플러그인의 실제 config 디렉터리와
기존 `env`·`.env` 파일을 확인합니다. 기본 config 위치는
`~/.config/herdr/plugins/config/devswha.herdr-web-ui/`입니다.

로컬 전용으로 사용할 때의 설정 예시는 다음과 같습니다. 실제 환경 파일은 Git에 넣지 않습니다.

```dotenv
HOST=127.0.0.1
PORT=7317
HERDR_WEB_AUTO_UPDATE=0
HERDR_WEB_TELEMETRY=0
```

포트 사용 여부와 해당 버전의 의존성을 확인한 뒤 설치·시작합니다.
기존 다른 컴퓨터의 인증정보나 Tailscale 주소·Serve 설정을 복사하지 않습니다.
이 저장소는 웹 UI를 외부에 공개하거나 Tailscale 설정을 변경하지 않습니다.

## 이후 설정 동기화하기

- 한 컴퓨터에서 설정을 변경하고, 검토한 공통 설정만 `herdr/config.toml`에 반영합니다.
- `git diff`로 내용을 확인하고 커밋·푸시합니다.
- 다른 컴퓨터에서는 `git pull --ff-only` 후 위의 검사·백업·적용 단계를 다시 실행합니다.
- 플러그인 ref를 바꾼 경우에만 해당 플러그인도 다시 설치합니다.
- Git pull만으로 실제 Herdr 설정이나 기존 로컬 플러그인 링크가 자동 변경되지는 않습니다.

## 공개 전 안전 확인

`.gitignore`는 기본적으로 모든 경로를 제외하고, 명시한 파일만 허용합니다.
환경 파일·인증 파일·세션·로그·백업·빌드 산출물에는 추가 제외 규칙도 적용합니다.
새 파일을 공유하려면 먼저 내용을 검토하고 허용 목록에 파일 경로를 추가합니다.

```sh
git status --short
git diff --cached --stat
git diff --cached
git ls-files
```

**`.gitignore`는 허용된 파일 안에 적은 비밀값까지 차단하지 않습니다.**
`config.toml`이나 소스에 토큰·내부 URL을 직접 추가하지 말고, 매번 변경 내용을 확인합니다.
`git add -f`는 제외 규칙을 우회하므로 사용하지 않습니다.
이미 추적 중인 파일이나 과거 커밋의 비밀값도 `.gitignore` 추가만으로 제거되지 않습니다.

## Agent Picker 검증

```sh
sh plugins/agent-picker/run check
sh plugins/agent-picker/run test
```

Picker 테스트는 실제 fzf와 가짜 Herdr 응답을 사용합니다.
실행 중인 Herdr 세션의 포커스를 바꾸지 않습니다.
집 컴퓨터에서의 설치·실행은 그 컴퓨터에서 별도로 확인해야 합니다.

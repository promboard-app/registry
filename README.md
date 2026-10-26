# PromBoard 플러그인 레지스트리

PromBoard 앱의 **플러그인 매니저**가 읽는 플러그인 목록입니다. 플러그인 파일(zip)은 각 개발자의 GitHub Releases 에 있고, 이 저장소에는 "어디서 받는지"만 적습니다.

> 이 폴더는 앱 저장소의 `server/registry-template/` 에서 복사한 본보기입니다. 검사 코드의 정본은 앱 저장소 `server/src/plugins/registry/` 이고, 형식 규칙은 `docs/Scrum4/02_스키마-v0.md` 가 소유합니다.

## 플러그인 올리는 법

1. **플러그인 저장소를 GitHub 에 만듭니다.**
   - 플러그인 id 는 `<GitHub 계정 이름>.<이름>` 입니다(영문 소문자·숫자·하이픈). 예: 계정 `kim-dev` → `kim-dev.remove-bg`
   - id 앞부분이 저장소 주인과 다르면 검사에서 떨어집니다. `promboard.` 는 공식 조직(`promboard-app`) 전용입니다.
2. **zip 을 만들어 Release 에 올립니다.**
   - zip **맨 위에** `manifest.json` 이 있어야 합니다. 폴더째 압축하지 마세요.
   - `__pycache__`·`.pyc`·실행 파일·wheel, zip 속의 또 다른 압축 파일은 넣지 마세요.
   - 파일 이름은 Windows 에서도 쓸 수 있어야 합니다. `aux`·`con`·`nul` 같은 예약 이름, 점이나 공백으로 끝나는 이름, 대소문자만 다른 두 파일은 안 됩니다. 파일은 1만 개까지입니다.
   - AI 모델 파일은 넣지 않습니다. `manifest.json` 의 `requires.models` 에 출처·커밋·해시만 적습니다.
3. **이 저장소에 PR 을 보냅니다.** 파일 하나, `plugins/<플러그인 id>.json` 을 추가합니다.

```json
{
  "schema": 1,
  "id": "kim-dev.remove-bg",
  "name": "배경 제거",
  "description": "이미지 배경을 지운다",
  "author": "Kim",
  "license": "MIT",
  "repository": "https://github.com/kim-dev/remove-bg",
  "contents": ["nodes"],
  "nodes": ["remove-bg"],
  "versions": [
    {
      "version": "0.1.0",
      "app": ">=1.0.0",
      "url": "https://github.com/kim-dev/remove-bg/releases/download/v0.1.0/kim-dev.remove-bg-0.1.0.zip",
      "bytes": 12345,
      "sha256": "<zip 의 sha256, 소문자 64자리>",
      "permissions": {}
    }
  ]
}
```

- **`url`** 은 `repository` 저장소의 `/releases/download/<태그>/<파일>.zip` 입니다. Release 화면의 파일 주소를 그대로 붙여 넣으세요. `..`·`%2e` 처럼 읽을 때 바뀌는 주소는 떨어집니다.
- **`versions[]` 의 `app` · `requires` · `permissions` · `nonCommercial` · `licenseNotice` · 라벨(`categories` · `languages` · `unverified` · `gpu` · `platforms`)** 은 zip 안 `manifest.json` 과 같아야 합니다(`platforms` 는 `python.lock` 에 적은 운영체제, 코드 없는 플러그인은 셋 다). 앱 저장소의 `server/src/tools/packPlugin.ts` 가 zip 과 항목을 같이 만들어 줍니다.
- **새 버전**: 배열 끝에 버전을 하나 더 붙입니다. **한 번 올린 버전은 고치거나 지울 수 없습니다.** 바꾸려면 버전을 올리세요.

## 아이콘 (`icons/<플러그인 id>.png`, 선택)

플러그인 매니저 카드와 상세 창에 보이는 그림입니다. 설치 전에도 보입니다. 같은 PR 이나 따로 낸 PR 로 `icons/<플러그인 id>.png` 을 더합니다.

- 정사각 PNG, 64~1024px(256px 권장), 256KB 이하. 모서리는 앱이 둥글게 자르니 네모 그대로 두고, 배경을 채웁니다.
- 회사 · 모델 로고를 쓸 때는 그 회사가 공개한 공식 로고를 씁니다.
- 아이콘이 없으면 zip 안 `manifest.json` 의 `icon` 그림을 설치 뒤에 보이고, 그것도 없으면 기본 퍼즐 아이콘입니다.

## 커스텀 노드 화면 (`manifest.json` 의 `nodes[]`)

노드 화면은 화면 코드 없이 칸 몇 개로 고릅니다. 모두 선택 칸이고, `schema` 번호는 1 그대로입니다. 아무것도 적지 않으면 앱이 출력 · 입력 개수와 데이터 타입에 맞춰 알아서 고릅니다(`auto`).

| 칸 | 값 | 뜻 |
|---|---|---|
| `layout` | `auto`(기본) · `single` · `dual` · `grid` · `sources` · `sourceStrip` · `settings` · `text` | `single` 출력 1칸 · `dual` 출력 2칸 · `grid` 격자(4칸, 넘치면 '+N') · `sources` 출력 + 소스 2칸 · `sourceStrip` 출력 + 소스 한 줄(가로 스크롤) · `settings` 출력 + 설정 패널 · `text` 글 상자 |
| `showSources` | `true` | 입력 원본을 소스 칸에 보입니다. `layout` 이 `auto` · `sources` · `sourceStrip` 일 때만 |
| `sources` | 입력 포트 `id` 목록(1~8) | 소스 칸에 보일 입력 포트. 안 적으면 앱이 `text` 밖 입력을 영상 → 그림 → 나머지 순으로 넣습니다. `text` 포트는 못 넣습니다 |
| `settingsPanel` | `true` | 설정을 모서리 알약 대신 오른쪽 패널로 보입니다. `layout` 이 `auto` · `settings` 일 때만 |
| `views` | `compare` · `resultText` · `frameStrip` · `orbit3d` | 레이아웃 위에 얹는 보기 방식. `compare` 전/후 비교 슬라이더(그림 입력 + 그림 출력) · `resultText` 결과 아래 글 상자와 [복사](글 출력 + 그 밖의 출력) · `frameStrip` 영상 프레임 줄(영상 출력) · `orbit3d` 3D 돌려 보기(3D 출력) |
| `compare` | `{ "before": "<그림 입력 id>", "after": "<그림 출력 id>" }` | 비교할 쌍. `views` 에 `compare` 가 있을 때만 쓰고, 안 적으면 첫 그림 입력과 첫 그림 출력입니다 |
| `renderEffect` | `audio` | 이 노드를 렌더 탭 효과로도 씁니다. 플러그인을 켜면 소리 클립 오른쪽 클릭 메뉴 [오디오 효과 ›]에 뜨고, 고른 클립 소리를 입력에 넣어 돌린 결과 소리로 바꿉니다(원래 소리로 되돌릴 수 있습니다). 입력 포트는 소리(`audio`) 하나(선 하나), 출력 포트도 소리 하나뿐이어야 합니다. 이 칸을 쓴 플러그인은 칸을 모르는 옛 앱이 건너뛰므로 `app` 을 `>=1.0.2` 이상으로 적으세요 |
| `template` | `image` · `video` · `imageRemix` · `memo` | 옛 칸입니다. `layout` 과 같이 쓰지 않습니다(`image` · `video` · `imageRemix` → `single`, `memo` → `text`). 옛 칸만 쓴 노드는 보기 방식(`views`)을 그리지 않으므로 `layout` 으로 바꿔 적으세요 |

다음 조합은 앱이 조용히 무시하므로 **검사에서 떨어집니다**.

- `layout` 과 `template` 을 같이 적음
- `single` 인데 글(`text`) 밖 출력 포트가 1개가 아님, `dual` 인데 2개가 아님, `text` 인데 `text` 출력이 없음
- 소스 칸 설정이 어긋남: 소스로 보일 입력(`text` 밖)이 없음, `sources` 에 없는 `id` 나 `text` 포트, `sources` · `sourceStrip` 인데 `showSources: false`, 소스 칸이 없는 레이아웃에 `showSources: true`
- 설정 패널 설정이 어긋남: `settings` 가 비어 있음, `settings` 레이아웃인데 `settingsPanel: false`, 설정 패널이 없는 레이아웃에 `settingsPanel: true`, 설정 패널과 `showSources` 를 같이 적음
- `text` 레이아웃 또는 옛 `template` 만 쓴 노드에 `views` · `compare` 를 적음, 보기 방식에 필요한 포트가 없음, `compare` 쌍이 그림 입력 · 그림 출력이 아님
- `renderEffect: "audio"` 인데 입력이 소리 하나(선 하나) · 출력이 소리 하나가 아님

## 라벨 (`manifest.json`)

매니저가 설치 전에 보여 주는 라벨입니다. 모두 선택 칸입니다.

| 칸 | 값 | 뜻 |
|---|---|---|
| `categories` | 1~3개: `imageGenerate` · `imageEdit` · `videoGenerate` · `videoAudio` · `motionTransfer` · `voice` · `music` · `sfx` · `model3d` · `upscale` · `photoEffect` · `pose` · `objectMask` · `audioEffect` | 기능 라벨(앞이 대표). 카드 알약과 기능별 거르기에 씁니다 |
| `languages` | 언어 태그 1~1000개(`ko` · `en` · `yue` · `zh-Hans` · `pt-BR`) | 목소리 · 음성 · 가사 모델의 지원 언어. 상세 창에 보입니다 |
| `unverified` | `true` | 만든 사람이 실제로 돌려 보지 못했습니다. '검증됨' 대신 '검증 안 됨'으로 보이고, 마우스를 올리면 그 까닭이 보입니다 |

- 고성능 · 서버급 알약은 칸이 없습니다. `gpu.vramMinMb` 가 16384 를 넘으면 고성능, 32768 을 넘으면 서버급입니다. 가장 가벼운 판이 실제로 도는 최소 값을 적으세요.
- 지원 OS 는 칸이 없습니다. `python.lock` 에 잠금 파일을 적은 운영체제(`win` · `mac` · `linux`)가 상세 창에 보이고, 코드 없는 플러그인은 셋 다입니다.
- 라벨 칸을 쓰면 `app` 을 `>=1.0.3` 이상으로 적으세요. 그보다 옛 앱은 이 칸을 몰라 플러그인을 건너뜁니다(검사에서 떨어집니다). 그런 버전은 항목에 `gpu` 와 `platforms` 도 옮겨 적습니다.

## 자동 검사 (PR)

`.github/workflows/check.yml` 이 PR 마다 확인합니다. 플러그인 코드는 **실행하지 않습니다**. 받아서 풀고 읽기만 합니다.

- 항목 형식, 파일 이름 = id, id 앞부분 = 저장소 주인
- 아이콘(`icons/`)은 레지스트리에 있는 플러그인의 `<id>.png` 이고 진짜 PNG · 정사각 · 크기 상한 안인가
- 올린 버전을 고치거나 지우지 않았는가
- 필요한 플러그인(`requires`)이 이 레지스트리에 있는가
- 새 버전의 zip 을 받아 크기·sha256 이 맞는가
- zip 안 `manifest.json` 이 항목과 같은가, 앱이 읽을 때 건너뛰지 않는가(카드·카드 세트·워크플로우, 카드가 가리킨 이미지·폰트·음원·라이선스 원문)
- zip 을 Windows 규칙으로 풀 수 있는가(예약 이름·끝 점·대소문자만 다른 이름 없음, 파일 1만 개까지)
- **라이선스**: 라이선스 종류로 플러그인을 거절하지 않습니다. 대신 사용자가 알 수 있게 조건을 적어야 합니다. 앱은 플러그인마다 이 안내를 보여 주고, 설치할 때 사용자에게 읽었다는 확인을 받습니다. 조건을 지키는 책임은 쓰는 사람에게 있습니다.
  - 비상업 라이선스(코드 · 카드 · 모델)가 하나라도 있으면 `manifest.json` 과 레지스트리 버전 항목에 `"nonCommercial": true` 를 적습니다. 적지 않으면 떨어집니다. 만든 결과물에는 '비상업' 표시가 붙습니다.
  - 라이선스가 일부 지역의 사용을 허락하지 않으면(예: Tencent Hunyuan 은 EU · 영국 · 대한민국, MiniMax-H3 는 여기에 미국까지) `"licenseNotice": { "excludedRegions": ["EU", "GB", "KR"] }` 처럼 적습니다. 지역은 ISO 국가 코드 두 글자와 `EU` 입니다. 이 두 라이선스는 검사가 알아봅니다(라이선스 이름 · 모델 저장소 · 모델 파일 주소 · AI 모델 endpoint, 호스팅을 거쳐도 같습니다). 적지 않으면 떨어집니다.
  - 매출 상한 · 표기 의무 같은 그 밖의 조건은 `licenseNotice.conditions`(500자까지), 원문 주소는 `licenseNotice.url` 에 적습니다.
  - 폰트 카드는 라이선스 원문(`.txt`·`.md`)을 같이 넣습니다. 폰트 파일을 플러그인에 담아 올리는 것은 재배포이므로, 재배포를 허락하는 폰트인지는 올리는 사람이 확인합니다.
- **재료 카드 파일**: 폰트(`ttf`·`otf`·`woff`·`woff2`)는 파일 하나 30MB, 음원(`mp3`·`wav`·`m4a`·`ogg`·`flac`)은 50MB 까지입니다. 확장자만이 아니라 파일 내용도 그 형식이어야 합니다.
- **금지 규칙**
  - 문자열을 코드로 실행(`eval`·`exec`·`compile`·`__import__`). 이름이 같은 메서드 정의(`def eval(self…)`)는 괜찮습니다
  - 숨긴 코드(`marshal`, 아주 긴 줄, 컴파일된 파일, utf-8 이 아닌 `# coding:` 선언)
  - 실행 중 패키지 설치(`pip install`), 셸 명령(`os.system`·`shell=True`)
  - `trust_remote_code=True`, `weights_only=False`
  - 넣을 수 없는 파일: 컴파일된 코드·실행 파일·패키지(`.pyc`·`.so`·`.dll`·`.exe`·`.sh`·`.whl` 등), 속에 든 압축 파일(`.zip`·`.egg`·`.pyz`·`.tar`·`.gz`·`.tgz`·`.7z`·`.rar`), `.pth`·`.pyw`
  - 코드 없는 플러그인 안의 코드 파일
  - 파이썬처럼 `\r` 하나만 쓴 줄바꿈도 줄로 나눠 봅니다. 주석 줄(`#`)은 보지 않습니다
- **커스텀 노드**: 적은 운영체제마다 잠금 파일이 있고, 모든 패키지 줄에 `--hash=sha256:` 이 있는가(`uv pip compile --generate-hashes` 결과 그대로면 됩니다). 진입 모듈이 있는가
  - 받는 곳(`--index-url`·`--extra-index-url`)은 `https://pypi.org/simple` 과 `https://download.pytorch.org/whl/…` 만 됩니다
  - 주소·로컬 경로·파일로 받는 패키지(`git+`·`https:`·`file:`·`@`·`./x.whl`)는 안 되고, 패키지 줄에는 해시 말고 다른 옵션을 붙일 수 없습니다
  - 줄 끝 `\` 만 다음 줄과 잇습니다. 주석이 든 줄을 `\` 로 잇거나 공백 없이 `\` 로 이으면 떨어집니다(pip 과 uv 가 다르게 읽습니다)
- **노드 화면 칸**: `layout` · `views` 등 노드 화면 칸(아래 「커스텀 노드 화면」)이 웹이 조용히 무시할 조합이면 떨어집니다
- **「시작 전에 할 일」 글**: 할 일 제목 · 기다림 안내 · 단계 글 · 링크 이름 · 키 이름에 줄바꿈이나 제어 문자가 있으면 떨어집니다
- **「시작 전에 할 일」 안내 링크**: 막지 않고 모아서 검사 결과의 🔗 줄(PR 검사 화면 알림)로 보입니다

자동 검사를 통과해도 **사람이 한 번 더 봅니다.** 금지 규칙은 글자 모양으로 찾는 검사라 빈틈이 있습니다(문자열·속성으로 우회, `subprocess` 목록 실행, `pickle` 등). 그래서 커스텀 노드는 **사람 심사가 꼭 필요합니다**. 남의 PC 에서 돌기 때문입니다.

AI 모델을 더하는 플러그인(`aiModels`)도 **사람 심사가 꼭 필요합니다**. 코드는 없지만 사용자 키로 요금이 나가고, 프롬프트 · 이미지가 그 모델로 가기 때문입니다. endpoint 가 설명과 같은 모델인지, 값이 지나치게 비싼 모델이 아닌지 봅니다.

사람 심사에서는 검사 결과의 🔗 줄(PR 검사 화면 알림)에 모인 안내 링크가 플러그인 설명과 맞는 곳인지도 확인합니다.

## registry.json

`.github/workflows/build.yml` 이 `main` 에 병합될 때와 하루 한 번 `registry.json` 을 다시 만듭니다. 목록에 Star 수와 다운로드 수를 채워 넣습니다. 사람이 직접 고치지 않습니다.

## 관리자 설정

- **조직 변수(선택)**
  - `PROMBOARD_APP_REPOSITORY`: 검사 코드가 든 앱 저장소. 기본값은 `guru99251/promboard`
  - `PROMBOARD_APP_REF`: 그 저장소의 브랜치·태그·커밋. 기본값은 `local/master`
- **앱 저장소가 비공개인 동안**
  - 비밀값 `PROMBOARD_APP_TOKEN`(앱 저장소 읽기 권한 토큰)이 필요합니다.
  - 포크에서 온 PR 에는 비밀값이 전달되지 않습니다. 그래서 공개 전까지는 **조직 멤버가 이 저장소의 브랜치로 PR** 을 보냅니다.
- **`main` 보호 규칙**(저장소 Rulesets): PR 필수 · 자동 검사(`check`) 통과 · 코드 소유자(`.github/CODEOWNERS`) 1명 승인. 예외는 배포 키(목록 갱신용)와, PR 병합 때의 조직 관리자뿐입니다.
- **목록 갱신 키**: `build.yml` 은 `registry.json` 을 `main` 에 직접 커밋합니다. GitHub 는 Actions 기본 계정을 보호 규칙 예외로 둘 수 없어서, 쓰기 권한 배포 키를 예외로 두고 그 비밀 키를 환경 `registry-build`(배포 브랜치 `main` 만)의 비밀값 `REGISTRY_DEPLOY_KEY` 에 넣습니다.

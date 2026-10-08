# PromBoard 플러그인 레지스트리

PromBoard 앱의 **플러그인 매니저**가 읽는 플러그인 목록입니다. 플러그인 파일(zip)은 각 개발자의 GitHub Releases 에 있고, 이 저장소에는 "어디서 받는지"만 적습니다.

> 이 폴더는 앱 저장소의 `server/registry-template/` 에서 복사한 본보기입니다. 검사 코드의 정본은 앱 저장소 `server/src/plugins/registry/` 이고, 형식 규칙은 `docs/Scrum4/02_스키마-v0.md` 가 소유합니다.

## 플러그인 올리는 법

1. **플러그인 저장소를 GitHub 에 만듭니다.**
   - 플러그인 id 는 `<GitHub 계정 이름>.<이름>` 입니다(영문 소문자·숫자·하이픈). 예: 계정 `kim-dev` → `kim-dev.remove-bg`
   - id 앞부분이 저장소 주인과 다르면 검사에서 떨어집니다. `promboard.` 는 공식 조직(`promboard-app`) 전용입니다.
2. **zip 을 만들어 Release 에 올립니다.**
   - zip **맨 위에** `manifest.json` 이 있어야 합니다. 폴더째 압축하지 마세요.
   - `__pycache__`·`.pyc`·실행 파일·wheel 은 넣지 마세요.
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

- **`versions[]` 의 `app` · `requires` · `permissions`** 는 zip 안 `manifest.json` 과 같아야 합니다.
- **새 버전**: 배열 끝에 버전을 하나 더 붙입니다. **한 번 올린 버전은 고치거나 지울 수 없습니다.** 바꾸려면 버전을 올리세요.

## 자동 검사 (PR)

`.github/workflows/check.yml` 이 PR 마다 확인합니다. 플러그인 코드는 **실행하지 않습니다**. 받아서 풀고 읽기만 합니다.

- 항목 형식, 파일 이름 = id, id 앞부분 = 저장소 주인
- 올린 버전을 고치거나 지우지 않았는가
- 필요한 플러그인(`requires`)이 이 레지스트리에 있는가
- 새 버전의 zip 을 받아 크기·sha256 이 맞는가
- zip 안 `manifest.json` 이 항목과 같은가, 앱이 읽을 때 건너뛰지 않는가(카드·카드 세트·워크플로우·이미지)
- **금지 규칙**
  - 문자열을 코드로 실행(`eval`·`exec`·`compile`·`__import__`)
  - 숨긴 코드(`marshal`, 아주 긴 줄, 컴파일된 파일)
  - 실행 중 패키지 설치(`pip install`), 셸 명령(`os.system`·`shell=True`)
  - `trust_remote_code=True`, `weights_only=False`
  - 코드 없는 플러그인 안의 코드 파일
- **커스텀 노드**: 적은 운영체제마다 잠금 파일이 있고, 모든 패키지 줄에 `--hash=sha256:` 이 있는가(`uv pip compile --generate-hashes`). 진입 모듈이 있는가

자동 검사를 통과해도 **사람이 한 번 더 봅니다.** 특히 커스텀 노드는 남의 PC 에서 돌기 때문입니다.

## registry.json

`.github/workflows/build.yml` 이 `main` 에 병합될 때와 하루 한 번 `registry.json` 을 다시 만듭니다. 목록에 Star 수와 다운로드 수를 채워 넣습니다. 사람이 직접 고치지 않습니다.

## 관리자 설정

- **조직 변수(선택)**
  - `PROMBOARD_APP_REPOSITORY`: 검사 코드가 든 앱 저장소. 기본값은 `guru99251/promboard`
  - `PROMBOARD_APP_REF`: 그 저장소의 브랜치·태그·커밋. 기본값은 `local/master`
- **앱 저장소가 비공개인 동안**
  - 비밀값 `PROMBOARD_APP_TOKEN`(앱 저장소 읽기 권한 토큰)이 필요합니다.
  - 포크에서 온 PR 에는 비밀값이 전달되지 않습니다. 그래서 공개 전까지는 **조직 멤버가 이 저장소의 브랜치로 PR** 을 보냅니다.
- **`main` 보호 규칙**: `build.yml` 이 `registry.json` 을 `main` 에 직접 커밋합니다. 그래서 `github-actions[bot]` 의 푸시를 허용해야 합니다.

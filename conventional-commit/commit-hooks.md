---
description: Commit Lint + PreCommit을 통한 코드 품질 유지하기!
---

# Commit Hooks

규칙을 정의하는 것과 그것이 지속될 수 있도록 하는 것은 별개의 문제이다.

커밋할 때 자동으로 컨벤션을 잘 지켰는지 검사하고, 실패 시 커밋 자체를 취소해 컨벤션을 지속적으로 지킬 수 있게 도와주는 절차가 필요하다고 생각했다.

추가적으로 커밋 전에 작성한 test코드를 실행시켜 코드 품질을 유지할 수 있는 절차가 있으면 더 완벽하다.

나는 이런 절차를 구성하기 위해 CommitLint와 pre-commit hook을 사용했다.

## CommitLint

CommitLint는 작성한 Commit Message가 컨벤션을 지켰는지 손쉽게 확인할 수 있도록 도와주는 라이브러리이다.

{% embed url="https://commitlint.js.org/#/" %}

### 설치 & 세팅

```
# 의존성 설치 
npm install -g @commitlint/cli @commitlint/config-conventional

# conventional commit 규약 대로 메시지를 검사하기 위해 프로젝트 루트 경로에 설정파일을 생성한다.
echo "module.exports = { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js
```

{% hint style="info" %}
node프로젝트는 husky를 사용했고, 그 외에는 수동으로   .git/hooks폴더에 commit-msg 훅을 추가했다.&#x20;
{% endhint %}



* **husky**

```shell
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

* **commit-msg**

```shell
#!/bin/sh
FONT_YELLOW="\033[33m"

echo "${FONT_YELLOW}>> Run commitlint before commit message"

(cat "$1") | commitlint

# 실행권한도 잊지 말고 추가하자. 
chmod 774 commit-msg
```



### 사용 해 보기

이제 컨벤션에 어긋난 메시지로 커밋을 하게 되면, 어디에서 오류가 발생했는지와 이유를 보여주고, 더 이상 커밋이 진행되지 않는다.

<figure><img src="../.gitbook/assets/스크린샷 2023-01-31 오후 3.55.45.png" alt=""><figcaption></figcaption></figure>



## pre-commit hook

pre-commit hook은 최종적으로 commit 되기 바로 직전에 발동하는 hook이다. 이 때 작성한 test 코드를 실행시키면 코드 품질을 유지할 수 있다. ci단에서 진한 빨간 에러를 보기 전에 고칠 수 있어 더욱 마음에 든다.&#x20;

이 경우도 husky면 쉽게 할 수 있으니 패스하고 아래는 go프로젝트를 예시로 간단히 테스트만 걸어놓은 pre-commit스크립트이다.&#x20;

```shell
// Some codeFONT_YELLOW="\033[33m"
BG_RED="\033[41m"

# set test flag here
GOTEST="go test -v ./..."

echo "${FONT_YELLOW}>> Run [ `echo ${GOTEST}` ] before commit."

${GOTEST}
if [ $? -ne 0 ]; then
echo "${BG_RED}>> Commit fail! Check your code.${NO_COLOR}"
exit 1
fi
```

### 사용해보기&#x20;



이제 커밋을 하면 test가 실행되며 케이스가 모두 통과되면 커밋이 완료 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-01-31 오후 4.37.49.png" alt=""><figcaption></figcaption></figure>

만약 테스트가 실패하면 커밋이 취소된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-01-31 오후 4.48.27.png" alt=""><figcaption></figcaption></figure>

### 정리&#x20;

* CommitLint Message를 사용해 커밋 메시지가 컨벤션을 지켰는지 검사할 수 있다.
* pre-commit hook을 이용해 커밋 전 테스트를 수행함으로써 해당 커밋이 어떤 사이드 이펙트로 오류를 야기 시켰는지 검사할 수 있다.&#x20;

다음은 Conventional Commit을 이용하면 장점인 버전을 자동화할 수 있는 방법을 소개한다.








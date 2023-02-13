# Conventional Commit

우리는 가금 바쁘거나 귀찮다는 이유로 커밋 메시지를 대충 작성하는 경우가 종종 있다. 그렇게 좋지 않은 커밋 메시지가 차곡차곡 쌓인 채로 훗날 커밋 히스토리를 보게 되면 아무리 혼자 작업한 프로젝트도 이해하기 어렵다.

어떻게 하면 좋은 커밋 메시지를 작성하고 유지할 수 있을지에 대해 고민하던 중, 코드 컨벤션처럼 커밋 메시지에도 컨벤션이 있다는 걸 알게 되었다.

### 좋은 Commit메시지란

해당 커밋 작업의 의도를 요약해서 잘 설명하는 것이 중요하다. 코드 설명 부분은 어떻게 보다는, 무엇이 변경 되었고, 왜 이렇게 구현 했는지에 대한 이유를 작성하는 것이 좋다고 한다. 나는 자신이 작업한 내용에 대한 이해도를 남들에게도 동기화하려는 노력을 커밋 메시지에 담으면 된다고 생각한다.

좋은 Commit 메시지를 남김으로써 우리는 동료들과 협업이 간소화할 수 있으며, 커밋 히스토리를 더 쉽고 빠르게 파악할 수 있을 것이다.

하지만 매번 의식하며 메시지 작성을 지속 하기란 힘들 것 같았다. 어떤 규칙을 정해 메시지 작성에 고민하는 시간을 줄여줄 무언가가 필요했다.&#x20;

### Conventional Commit

Conventional Commit은 [Angular Commit Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)에 강한 영감을 받아 작성된 Commit Message 명세이다. Conventional Commit을 통해 메시지 작성마다 메시지 포맷을 매번 고려해야 하는 비용을 없애주고, 더 쉽게 커밋과 커밋 히스토리를 파악할 수 있으며, 명세된 메시지들은 [Semantic Version](https://semver.org/)에 대응이 되기 때문에, 메시지를 참고해 Semantic Version이나 CHANGELOG를 자동으로 작성할 수 있다.

### 메시지 작성법

```
[타입]([적용범위]): [주제]
<빈 칸>
[본문]
<빈 칸>
[꼬리말]

--------------------------------------------------------------------------------------------------------

ex)
feat(user-service): add reset user password service

비밀번호를 까먹는 유저들의 문의가 점점 많아져 비밀번호 초기화 기능 추가

Closes: #MERGY-1519
```

위의 형식대로 \[타입], \[적용범위] , \[주제], \[본문], \[꼬리말]에 알맞는 요소를 작성하면 된다.&#x20;



### 요소 소개&#x20;

#### **타입 (필수)**

어떤 종류의 작업을 했는지에 대해 명시한다.

* feat: 기능 추가/수정 등 => Semantic Version에 MINOR에 대응한다.
* fix: 버그 수정 => Semantic Version에 PATCH에 대응한다.
* BREAKING CHANGE: 주요 변경점으로 기존 API가 호환되지 않을 수 있다. 따라서 MAJOR에 대응된다.
* refactor: 리팩토링
* test : 테스트
* chore : 빌드, 패키지 관련 (업데이트 등) => 버전이 업그레이드 되지 않는다.
* docs : 문서 수정
* style: 스타일 변경 (형식, 세미콜론 누락 등)
* revert: revert작업
* ci: CI 환경설정

#### **적용범위(선택사항)**

작업코드가 적용되는 곳을 명시한다. ex) api, util, user-service

#### **주제 (필수)**

코드에 대해 요약해서 설명한다.\


#### **본문(선택사항)**

어떻게 보다는 왜, 무엇을 했는지를 적는 것이 좋다. \


#### **꼬리말(선택사항)**

commit에 대한 메타데이터를 작성한다. 아래와 같은 메타데이터를 추가할 수 있다.

* Closes: 이슈를 해결했을 때
* Refs: 참고할 이슈가 있을 때
* Co-authored-by: 공동 작업자





이상으로 Conventional Commit에 대해 알아보았다. 처음 사용할 땐 포맷과 필요한 요소가 햇갈릴 수 있으니 IDE에서 사용할 수 있는 plugin을 추천한다.&#x20;

[Conventional Commit - IntelliJ IDEs Plugin | Marketplace](https://plugins.jetbrains.com/plugin/13389-conventional-commit)



<figure><img src="../.gitbook/assets/스크린샷 2023-02-01 오전 11.35.03.png" alt=""><figcaption><p>메지지 빌더</p></figcaption></figure>

다음 글은 컨벤션을 지속적으로 유지할 수 있게 도와주는 기술에 대해 알아보려 한다.



**참고**

[Conventional Commits](https://www.conventionalcommits.org/ko/v1.0.0-beta.4/#%ea%b0%9c%ec%9a%94)

[Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1\_OOAqWjiDU5Y/edit)

[\[Git\] 커밋 메시지를 잘 작성해보자!](https://velog.io/@ozragwort/Git-%EC%BB%A4%EB%B0%8B-%EB%A9%94%EC%8B%9C%EC%A7%80%EB%A5%BC-%EC%9E%98-%EC%9E%91%EC%84%B1%ED%95%B4%EB%B3%B4%EC%9E%90#%EC%BB%A4%EB%B0%8B-%EB%A9%94%EC%8B%9C%EC%A7%80%EC%9D%98-%EC%BB%A8%EB%B2%A4%EC%85%98)

[Commit History를 효과적으로 관리하기 위한 규약: Conventional Commits](https://medium.com/hdackorea/commit-history%EB%A5%BC-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0-%EC%9C%84%ED%95%9C-%EA%B7%9C%EC%95%BD-conventional-commits-67b2114ac8e4)\

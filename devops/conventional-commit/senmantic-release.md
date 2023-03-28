---
description: Senmantic Release를 통해 자동으로 version과 changelog를 업데이트하자.
---

# Senmantic Release



저번 글에서 Conventional Commit을 이용한다면 senmantic version(semver)과 changelog를 자동으로 업데이트 할 수 있다고 했다.&#x20;

Senmantic Release 라이브러리를 CI단에서 사용해서 실제로 어떻게 자동으로 업데이트 할 수 있는지 알아보자.



## Senmantic-Release

Conventional Commit에서는 해당 커밋이 어떤 타입인지를 명시한다. feat, fix, breaking-change등의 타입은 semver에서 patch, minor, major에 대응한다. 따라서 기존 version과 커밋 메시지 타입을 토대로 다음 version을 계산할 수 있다.

Senmantic-Release는 이런 작업을 편하게 해주는 라이브러리로 새로운 semrver를 추출할 뿐 아니라 chnagelog, tag도 같이 업데이트 푸시 해줄 수 있다.&#x20;

나는 gitlab-ci환경에서 동작하는 것 확인했다.



## 세팅하기

### 1. 설정파일 &#x20;

저장소 최상위 경로에 .releaserc.json 설정파일을 생성했다. master branch에서 적용되며 명시한 plugin을 사용한다는 뜻이다.&#x20;

```json
{
  "branches": [
    "master"
  ],
  "plugins": [
    "@semantic-release/commit-analyzer", // 커밋 메시지를 분석 
    "@semantic-release/release-notes-generator", // changelog 내용 추출 
    "@semantic-release/changelog", // chnagelog 생성, 업데이트
    "@semantic-release/gitlab", // gitlab과 연동 
    "@semantic-release/git" // git
  ]
}
```

<details>

<summary>plugin git url</summary>

[https://github.com/semantic-release/commit-analyzer](https://github.com/semantic-release/commit-analyzer)

[https://github.com/semantic-release/release-notes-generator](https://github.com/semantic-release/release-notes-generator)

[https://github.com/semantic-release/changelog](https://github.com/semantic-release/changelog)



</details>



### 2. Git Token 준비하기

git repository에 접근할 수 있게 access\_token을 준비한다. tag와 changelog를 푸시할 수 있도록 토큰에 적절한권한을 주도록 하자.&#x20;

[Personal access tokens | GitLab](https://docs.gitlab.com/ee/user/profile/personal\_access\_tokens.html)

[https://github.com/settings/tokens](https://github.com/settings/tokens)

###

### 3. CI 환경변수 설정&#x20;

나는 GL\_TOKEN이라는 키에 위에서 만든 TOKEN값을 넣었다.&#x20;



### 4.  CI 설정

아래는 간소화한 ci파일이다. versioning 스테이지는 테스트 스테이지에서 테스트가 성공해 코드에 문제가 없다는 것을 검증한 뒤 실행한다.&#x20;

```yaml
stages:
  - lint
  - install
  - test
  - versioning
  - build
  - deploy

variables:
  PROJECT_NAME: semantic-versioning-example-go
  VERSION_FILE: ./version.txt # 버전 정보를 다음 스테이지에 넘기기 위해서.

lint:
  stage: lint
  script:

install:
  stage: install
  script:
        
test:
  stage: test
  script:
      - go test


semantic-versioning:
  image: smartive/semantic-release-image # 
  stage: versioning
  variables:
    GL_TOKEN: $GL_TOKEN # Git AccessToken을 넣어준다. 
  script:
    # semnatic-release가 작동해 버전과 changelog를 추출하고 tag를 푸시한다.
    - npx semantic-release 
  only:
    - master
  after_script:
  # 업데이트된 버전을 다음 스테이지에 넘겨준다. 
    - echo "after versioning"
    - latest_version=$(git describe --tags --abbrev=0)
    - echo $latest_version
    - echo "export latest_version=$latest_version" > $VERSION_FILE
  artifacts:
    paths:
      - $VERSION_FILE


build:
  image:
    name: registry.gitlab.com/gitlab-org/cloud-deploy/aws-base:latest
  stage: build
  only:
    - master
  script:
    - source $VERSION_FILE # 버전 파일에 입력된 업데이트 된 버전을 가져온다.
    - echo $latest_version
    - docker build -t $PROJECT_NAME:$latest_version .
    - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY
    - docker tag $PROJECT_NAME:$latest_version $ECR_REGISTRY/$PROJECT_NAME:$latest_version # 이후 ecr tag에도 적용한다.
    - docker push $ECR_REGISTRY/$PROJECT_NAME:$latest_version


deploy-prod:
  stage: deploy
  only:
    - master
  script:
    - echo "deploy"
```



## 결과

ci가 정상적으로 돌아가면서 semantic-release가 동작되면서 1.0.0에서 최종적으로 1.0.1로 버저닝이 된 것을 볼 수 있다.&#x20;

첫 커밋인 feat은 마이너이기 때문에 0.1.0을 기대했지만 기존 버전 정보가 없다면 1.0.0으로 되는 것을 확인했다. 이 후 fix 커밋 메시지이기 때문에 patch버전이 +1되어 1.0.1로 최종 업데이트 되었다.

또 각 버전 업데이트에 뒤를 이어 changelog와 tag를 푸시하는 것도 볼 수 있다. changelog와 tag push작업은 chore타입으로 ci/cd 파이프라인을 스킵한다.

<figure><img src="../../.gitbook/assets/스크린샷 2023-01-31 오후 5.52.33.png" alt=""><figcaption></figcaption></figure>

태그가 잘 업데이트 되었다.&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2023-01-31 오후 5.54.14.png" alt=""><figcaption><p>Tag</p></figcaption></figure>

CHANGELOG.md 파일도 자동으로 작성해준다.

<figure><img src="../../.gitbook/assets/스크린샷 2023-01-31 오후 5.53.59.png" alt=""><figcaption><p>CHANGELOG.md</p></figcaption></figure>



## 마치며&#x20;

깃 커밋 메시지에 대해서 알아보다가 어쩌다 보니 버저닝 자동화까지 알게 되었다. 실제로 적용하고 사용해보니 편하고 유용한 것 같다. conventional commit을 적용하는 프로젝트에는 Senmantic Release도 적용할 예정이다.


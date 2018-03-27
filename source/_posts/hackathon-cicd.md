---
title: 해커톤 나가서 삽질하기(2) - CI / CD 환경 구축(FE)
date: 2018-03-23 12:48:23
categories:
    - devops
    - hackathon
    - travis
    - react
---

### 해커톤 나가서 삽질하기(2) - CI / CD 환경 구축(FE)

> 들어가기에 앞서 저에게 늘 새로운 인사이트를 주시고 실무적인 테크닉도 넘치게 전달해 주시는 창천향로님의 블로그([http://jojoldu.tistory.com](http://jojoldu.tistory.com))와 창천향로님께 무한한 감사의 말씀을 드린다. 
>
> 오늘 소개할 내용도 많은 부분이 창천향로님의 "스프링 부트로 웹서비스 출시하기" 시리즈에서 배운 것을 활용해보고 소개하는 것이다. 

**결론 - 해커톤 나가서 괜히 CICD 한다고 삽질하지 말자. 전혀 의미없다.**

나는 이번 해커톤에 참가하며 부족한 데브옵스(DevOps) 및 서버 운영 지식을 보충하겠다는 확고한(?) 목적 의식을 갖고 있었다. 

때문에 CI / CD 환경 구축을 위한 초기 환경 세팅(Travis 가입, EC2 인스턴스 기본 세팅, 슬랙 채널 생성)을 마친 상태였다. **이 작업에 만 하루 정도, 순작업시간 4~5시간 정도**가 소요되었다.

물론 처음 해본 일이고, EC2 과금을 피하기 위한 눈물겨운 사투(...)를 벌였기 때문에 시간이 길어진 감이 있는데 *24시간 해커톤에서 4시간을 날리는 것은 엄청난 손해임이 분명하다.* MVP를 개발해 보여주는게 목표인 해커톤에서는 localhost:8080 해도 괜찮다. 

#### create-react-app 로컬 개발환경에서의 빌드와 배포

**간단하다!** 아래 명령어만 입력하면 되는 거 아닌가. 

```bash
yarn build

(혹은....)

npm run build
```

#### Travis CI - Amazon S3 환경에서의 빌드와 배포

S3 Bucket의 정적 사이트 호스팅 기능을 사용하고, Travis CI를 통해 빌드 후 배포한다고 가정해 보겠다. 각 서비스의 **기본 세팅이나 버킷 생성 등 준비절차** 는 설명하지 않는다. 

Flow는 다음과 같다.
> Master에 push/merge -> Travis VM Build -> S3 Upload

* .travis.yml 작성

```yaml
language: node_js
node_js:
  - "stable"
cache:
  directories:
  - node_modules
script:
  - yarn test
  - yarn build

deploy:
  provider: s3
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key: $AWS_SECRET_KEY
  bucket: YOUR-BUCKET
  region: ap-northeast-2 #서울의 리전명. 
  local_dir: build #build 후 ./build 디렉토리만 올라가게 한다.
  skip_cleanup: true

notifications:
  slack: YOUR-SLACK-KEY
```

React 기반의 FE CI / CD는 이걸로 끝이다. **참 쉽죠?**


---
title: "2. Travis CI"
date: 2020-05-15
weight: 4
description: >

#   A short lead descripton about this content page. It can be **bold** or _italic_ and can be split over multiple paragraphs.
---

## apptest.ai Integration for Travis CI

몇 줄의 스크립트만으로 apptest.ai를 Travis CI와 함께 사용하여 testbot에서 앱을 탐색하고 모든 빌드 프로세스에 대한 결과를보고 할 수 있습니다.

Travis CI 파이프 라인 내에서 apptest.ai 서비스를 사용하려면 다음 두 단계를 수행하십시오. 1. apptest.ai의 액세스 키를 등록하고 2. apptest.ai를 빌드 스크립트 (travis.yml)에 추가하십시오.

#### Adding apptest.ai access key onto the build environment

- apptest.ai 서비스의 Profile 페이지에서 액세스 키 문자열을 복사하십시오.
  {{< figure src="../../../images/2_travisci_1.gif" >}}

* APPTEST_AI_ACCESS_KEY라는 빌드 환경 변수에 apptest.ai 액세스 키를 추가하고 빌드 스크립트에 암호화를 추가하십시오.

  ```
  $ travis login
  $ travis encrypt APPTEST_AI_ACCESS_KEY="ID:Access Key"  --add
  ```

  > 액세스 키의 형식은 "apptest.ai ID : apptest.ai 액세스 키"가됩니다. 예를 들어, ID가 jean@apptest.ai이고 액세스 키가 "ab8f3e321d631c84c9b1113"인 경우 다음이 적용됩니다.

  ```
  jean@apptest.ai:ab8f3e321d631c84c9b1113
  ```

  이 명령 후에 다음“env :”섹션이 기존 빌드 스크립트 파일에 추가됩니다. 이 섹션은 런타임시 APPTEST_AI_ACCESS_KEY 환경 변수 선언으로 대체됩니다.

  ```
  env:
    global:
    - secure: fUnmuuqKmxWjxGHbfTB7QZgSmkB7yvt6KGAn8dCs1j4GneWbSIrEhghRrlOMQCThhMoetou4H+NaWeTXRRa4fpoCGm1U51RbD+J4DrbKWj3MzRupzByEAsaWhmvCaYVgCOlkYgaApcvujIrpw1rUTQVio8+WoYIXgXx3B
  ```

* 이제 apptest.ai에서 설정한 CI 도구 통합 스크립트를 다음과 같이 복제 한 후 다음을 실행하는 스크립트를 추가하십시오.

  ```
  - git clone https://github.com/apptestai/ci_addon
  - export binary_path=HackerNews.ipa
  - export project_id='825'
  - bash ./ci_addon/step.sh
  ```

  > 서비스를 호출하려면 최소한 2 개의 arguments, binary_path 및 project_id가 필요합니다. binary_path는 테스트하려는 바이너리 파일의 경로입니다 (예 : Android 용 _ .apk 파일 및 iOS 용 _ .ipa 파일). project_id는 테스트 시간 및 테스트를 위한 장치 목록과 같은 설정을 포함하는 apptest.ai의 프로젝트 고유번호(Project ID)입니다. apptest.ai의 프로젝트 ID는 아래와 같이 프로젝트 정보 페이지에서 찾을 수 있습니다.

  {{< figure src="../../../images/2_travisci_2.png" >}}

apptest.ai의 CI 통합 스크립트를 매번 다운로드하지 않으려면 프로젝트 저장소에 apptest.ai의 ci_addon 저장소를 미리 포함하십시오.

- 빌드 스크립트를 커밋하면 Travis CI가 실행됩니다. apptest.ai 스크립트를 실행할 때 아래와 같이 테스트가 실행되고 결과가 콘솔에 인쇄됩니다.
  {{< figure src="../../../images/2_travisci_3.png" >}}

* 테스트 결과는 자세한 HTML 형식의 보고서는 test-results/apptest-ai_result.html에 JUnit 테스트 결과 형식의 XML 문서는 test-results/apptestai/result.xml에 저장됩니다. Travis CI에서 제공하는 다양한 배포 기능을 사용하여이 파일을 외부에 저장할 수 있습니다. 아래는 Github의 릴리즈 예입니다. apptest.ai 테스트 보고서는 Github의 릴리스 페이지에 게시되어 있습니다.
  {{< figure src="../../../images/2_travisci_4.gif" >}}

> 결과를 저장할 폴더의 위치를 변경하려면 test_result_path 환경 변수를 선언하고 전달하십시오. 테스트가 완료되기를 기다리지 않고 다음 빌드를 실행하려면 waiting_for_test_results 환경 변수를 선언하고 FALSE를 전달하십시오.

마지막으로 .travis.yml 예제가 있습니다. 먼저 Android의 "env"섹션에서 apptest.ai 액세스 키를 저장하기 위한 두 가지 보안 환경 변수와 Github access 키를 배포하기 위한 두 가지 보안 환경 변수가 있음을 알 수 있습니다.
{{< gist genie-park afb71bfb6981edbc109eb5b8511007de >}}

iOS의 경우 iOS에는 등록 된 환경 변수가 두 개 더 필요합니다. 이것은 iOS에서 바이너리를 생성 할 때 필요한 Code sign을 위한 환경 변수입니다. 아래에서는 Code Sign에 Fastlane 매칭을 사용했습니다. Fastlane match는 Github에서 코드 서명에 필요한 인증서 파일과 프로비저닝 파일을 암호화하므로 인증서가 저장된 개인 GitHub 리포지토리에 액세스하기 위해 두 개의 액세스 토큰과 암호 해독 키가 추가됩니다.

{{< gist genie-park 8c3756623d48b472e177bc473c5c9338 >}}

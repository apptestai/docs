---
title: "3. Circle CI"
date: 2020-05-15
weight: 4
description: >

#   A short lead descripton about this content page. It can be **bold** or _italic_ and can be split over multiple paragraphs.
---

## apptest.ai Integration for Circle CI

CircleCI 버전 2.1부터 orb라는 빌드 스크립트 공유 방법을 사용하게 됨으로써 단 몇 줄의 스크립트만으로 매 빌드마다 Testbot이 앱을 테스트한 결과를 보고받을 수 있습니다.

CircleCI에서 빌드하는 동안 apptest.ai 서비스를 활성화하려면 : 1. 빌드 환경 변수에 apptest.ai 액세스 키를 등록합니다.

#### Adding apptest.ai access key onto the build environment

1. apptest.ai 서비스의 Profile 페이지에서 액세스 키 문자열을 복사하십시오.
   {{< figure src="../../../images/3_circleci_1.gif" >}}

2) CircleCI에 연결하고 빌드 프로젝트 이름 오른쪽의 설정 아이콘을 클릭하여 빌드 구성 페이지로 이동하십시오.
   {{< figure src="../../../images/3_circleci_2.png" >}}

3. 설정 페이지에서 환경 변수 메뉴를 선택하고 변수 추가 단추를 클릭하여 APPTEST_AI_ACCESS_KEY 변수를 작성하십시오.
   {{< figure src="../../../images/3_circleci_3.png" >}}

변수 값을 "apptest.ai ID : apptest.ai 액세스 키"로 입력하십시오. 예를 들어 ID가 jean@apptest.ai이고 액세스 키가 "ab8f3e321d631c84c9b1113"인 경우입니다. 다음과 같이 등록하십시오.

```
   jean@apptest.ai:ab8f3e321d631c84c9b1113
```

다음은 일반적인 등록이며 access_key는 마스크 처리되고 출력됩니다.

#### Adding apptest.ai test step to the build

1. Include apptest.ai Orb

   테스트하려는 앱이 Android 인 경우 아래와 같이 android-app-test orb를 포함하십시오.

   ```
   orbs:
      apptest-ai: apptestai/android-app-test@volatile
   ```

   iOS 앱을 테스트하려면 아래 표시된대로 ios-app-test orb를 포함하십시오.

   ```
   orbs:
      apptest-ai: apptestai/ios-app-test@volatile
   ```

2. Adding apptest.ai test build step

   빌드 워크 플로우의 어느 곳에나 apptest-ai 테스트 단계를 추가 할 수 있습니다.

   ```
   - apptest-ai/run-test:
      binary_path: app/build/outputs/apk/debug/app-debug.apk
      project_id: '818'
   ```

   > 서비스를 호출하려면 최소한 2 개의 arguments, binary*path 및 project_id가 필요합니다. binary_path는 테스트하려는 바이너리의 경로입니다. * Android의 경우 .apk 파일 및 iOS의 경우 \_ .ipa 파일입니다. project_id는 테스트 시간 및 테스트 할 장치 목록과 같은 설정이 포함 된 apptest.ai의 프로젝트 고유번호 (Project ID)입니다. apptest.ai의 프로젝트 ID는 아래와 같이 프로젝트 정보 페이지에서 찾을 수 있습니다.
   > {{< figure src="../../../images/3_circleci_4.png" >}}

3. Execute the build

   apptest-ai/run-test 단계는 apptest.ai로 테스트 요청을 보내고 테스트가 완료된 후 테스트 결과를 아래 그림과 같이 콘솔에 인쇄합니다.
   {{< figure src="../../../images/3_circleci_5.png" >}}

> apptest.ai 테스트 결과를 기다리지 않고 다음 빌드 단계로 넘어가려면 아래와 같이 waiting_for_test_results 인수를 FALSE로 설정하면 됩니다.

```
- apptest-ai/run-test:
   binary_path: app/build/outputs/apk/debug/app-debug.apk
   project_id: '818'
   waiting_for_test_results: 'false'
```

4. (Optional) Storing test result

   콘솔에 출력 된 apptest.ai 테스트 결과는 기본적으로“test-results”디렉토리에 저장됩니다. CircleCI에서 제공 한 테스트 요약에 테스트 결과를 포함 시키려면 CircleCI에서 제공 한 store-test-results 단계를 추가하고 아래와 같이 apptest.ai의 테스트 결과 경로를 지정하십시오.

   ```
   - store_test_results:
      path: test-results
   ```

   Store-test 결과 단계를 추가하면 CircleCI는 아래와 같이 Test Summary 탭에 apptest.ai 테스트 결과와 다른 테스트 결과를 분석합니다.
   {{< figure src="../../../images/3_circleci_6.png" >}}

5. (Optional) Storing test result

   테스트 요약은 매우 제한된 통과 / 실패 정보 만 제공하며 apptest.ai는 보다 자세한 테스트 결과 보고서를 별도의 HTML 형식으로 제공합니다. 이 보고서를 CircleCI의 아티팩트 섹션에 저장하려면 다음과 같이 CircleCI에서 제공 한 store_artifacts 단계를 추가하고 apptest.ai의 테스트 결과가 저장되는 경로를 지정하십시오.

   ```
   - store_artifacts:
      path: test-results
   ```

   저장된 테스트 보고서는 CircleCI의 아티팩트 탭에서 아래와 같이 조회됩니다.
   {{< figure src="../../../images/3_circleci_7.png" >}}

이 html 파일을 클릭하면 아래와 같은 상세 보고서를 볼 수 있으며,이 html 파일을 클릭하면 아래와 같은 상세 보고서를 볼 수 있습니다.
{{< figure src="../../../images/3_circleci_8.png" >}}

> 테스트 결과를 저장하는 디렉토리를 변경하려면 apptest.ai 테스트 단계를 다음과 같이 실행할 때 test_result_path 인수에 경로를 전달할 수 있습니다.

```
- apptest-ai/run-test:
   binary_path: app/build/outputs/apk/debug/app-debug.apk
   project_id: '818'
   test_result_path: app/build/outputs/apk/debug
```

마지막으로 최종 .circleci / config.yml 파일 샘플이 있습니다. 아래는 안드로이드 용입니다.

{{< gist genie-park cc0804ab72097afffc718defe69b83e9 >}}

iOS:

{{< gist genie-park 078f9d25e81f75a2d61435c3684367f9 >}}

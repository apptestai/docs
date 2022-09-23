---
title: "1. Jenkins"
date: 2020-05-15
weight: 4
description: >

#   A short lead descripton about this content page. It can be **bold** or _italic_ and can be split over multiple paragraphs.
---

## Jenkins 사용자를 위한 apptest.ai 통합 가이드

##### Jenkins 파이프라인으로 Android 및 iOS 앱 테스트 자동화하기

이 문서는 Jenkins를 구성하고 API를 사용하여 빌드 단계에서 apptest.ai 테스트를 자동으로 실행할 수 있도록 통합시키는 방법을 설명합니다.

Jenkins 설치에 대해서는 [Jenkins Setup Guide](https://jenkins.io/doc/pipeline/tour/getting-started/) 링크를 참조하시기 바랍니다.

### 1. Apptest.ai – Integration API

자세한 내용은 Rest APIs for CI/CD 문서를 참고해 주세요.

<div style="margin-bottom:20px;">
    <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> POST </span>
    <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">https://api.apptest.ai/openapi/v2/testset</span>
</div>
<div style="margin-bottom:20px;">
    <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> Authorization </span>
    <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">Basic {user_id}:{access_key}</span>
</div>

File Parameters

| Name     | Type | Required / Optional | Description                                                      |
| :------- | :--- | :------------------ | :--------------------------------------------------------------- |
| apk_file | File | Required            | 테스트하기위한 App binary file </br>Automated test일 경우 필수\* |

JSON Data parameters

| Name         |          | Type             | Required / Optional | Description                                                                                                                              |
| :----------- | :------- | :--------------- | :------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| pid          |          | Integer          | Required            | apptest.ai에서 생성한 프로젝트의 고유번호                                                                                                |
| testset_name |          | String           | Required            | apptest.ai의 프로젝트에 생성될 테스트 세트의 이름                                                                                        |
| scenario_id  |          | Number           | Optional            | 시나리오 고유번호 ex) 509”                                                                                                               |
| source_type  |          | String           | Optional            | 시나리오테스트 전용 테스트 대상의 종류 (앱 파일 또는 앱 아이디) 프로젝트의 앱 저장소에 설정된 대상의 종류 시나리오 테스트 진행 시 필수\* |
| os_type      |          | String           | Optional            | 테스트 대상의 os type                                                                                                                    |
| time_limit   |          | Positive Number  | Optional            | 테스트 시간 제한(분)이 비어 있으면, 프로젝트 설정에 저장된 시간 제한을 따릅니다.                                                         |
| credentials  | login_id | String           | Optional            | Testbot이 로그인 화면을 만났을때 입력할 앱의 로그인 아이디                                                                               |
|              | login_pw | String           | Optional            | Testbot이 로그인 화면을 만났을때 입력할 앱의 로그인 비밀번호                                                                             |
| use_vo       |          | Integer (0 or 1) | Optional            | AT&T사의 Video Optimizer (ARO)를 활성화 / 비활성화하는 옵션                                                                              |
| callback     |          | String           | Optional            | 테스트가 완료되면 호출될 Callback URL                                                                                                    |

Example Response

```
{
  "data": {
    "test_count": 1,      // Test Count (Integer),
    "testset_id": 192948        // Test Set ID (Integer)
  },
  "errorCode": 0,
  "reason": "",
  "result": "ok"
}
```

Callback result data format

Callback URL을 통해 리턴되는 JUnit XML 형식의 테스트 결과

```
<?xml version="1.0" encoding="UTF-8"?>
<testsuites name="TestBot Test">
    <testsuite name="TestBot Test.Apk File Name (with Version)">
      <testcase name="Device Name 1" time="Test Duration (sec)">
      </testcase>
      <testcase name="Device Name 2" time="Test Duration (sec)">
        <error message="apptest.ai Result Page Link"></error>
      </testcase>
      ...
    </testsuite>
</testsuites>
```

테스트 상태 조회 API

- <div style="margin-bottom:20px;">
      <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> GET </span>
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">https://api.apptest.ai/openapi/v2/testset/{testset_id}</span>
  </div>
- <div style="margin-bottom:20px;">
      <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> Authorization </span>
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">Basic {user_id}:{access_key}</span>
  </div>

테스트 결과 조회 API

- <div style="margin-bottom:20px;">
      <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> GET </span>
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">https://api.apptest.ai/openapi/v2/testset/{testset_id}/result</span>
  </div>
- <div style="margin-bottom:20px;">
      <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> Authorization </span>
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">Basic {user_id}:{access_key}</span>
  </div>

시나리오 아이디 목록 조회 API

- <div style="margin-bottom:20px;">
      <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> GET </span>
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">https://api.apptest.ai/project/{project_id}/scenarios</span>
  </div>
- <div style="margin-bottom:20px;">
      <span style="background-color: #5cb85c;padding: 5px 10px;border-radius: 5px;color: #fff;margin-right: 10px;"> Authorization </span>
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px 10px;border-radius: 5px;">Basic {user_id}:{access_key}
  </span>
  </div>

### 2.1 apptest.ai – Access Key and Project ID

apptest.ai를 Jenkins 파이프 라인에 통합하려면 액세스 키와 프로젝트의 고유번호(Project ID)가 필요합니다.

액세스 키를 찾는 방법 : apptest.ai에 가입하면 액세스 키가 자동으로 발급됩니다. apptest.ai Profile 페이지에서 확인하실 수 있습니다.
{{< figure src="../../../images/new/new_jenkins_1.png" >}}
{{< figure src="../../../images/new/new_jenkins_2.png" >}}

프로젝트 ID를 찾는 방법 : 테스트 프로젝트를 만들 때 프로젝트 ID가 할당됩니다.
{{< figure src="../../../images/new/new_jenkins_3.png" >}}
{{< figure src="../../../images/new/new_jenkins_4.png" >}}

회원가입시 기본적으로 샘플 테스트 프로젝트가 1개 생성됩니다.

샘플 테스트 프로젝트의 설정 변경은 지원되지 않습니다. 그러나 새로 생성한 프로젝트는 설정을 변경할 수 있습니다.

### 2.2 apptest.ai – 앱 저장소 및 시나리오

시나리오 테스트는 apptest.ai 에서 제공하는 시나리오 저작도구인 Stego를 통해 작성된 시나리오만 플레이 가능합니다.

또한 Jenkins와 연동하여 시나리오 테스트 CI 를 구성하려면 apptest.ai 서비스에 앱과 시나리오가 등록되어 있어야 합니다.

• 앱 등록 방법

앱 등록은 apptest.ai 서비스의 프로젝트 설정 페이지에서 가능합니다.
파일 혹은 App ID (Package name, Bundle ID) 를 등록할 수 있습니다.

{{< figure src="../../../images/new/new_jenkins_5.png" >}}
{{< figure src="../../../images/new/new_jenkins_6.png" >}}
{{< figure src="../../../images/new/new_jenkins_7.png" >}}

• 시나리오 등록 방법

시나리오등록은 Apptest.ai 서비스의 프로젝트 설정 페이지에서 가능합니다.

시나리오를 등록하기 위해서는 먼저 앱 저장소에 앱을 등록해야 합니다.

등록된 시나리오의 ID number를 이용해 시나리오 테스트를 진행할 수 있습니다.

{{< figure src="../../../images/new/new_jenkins_8.png" >}}
{{< figure src="../../../images/new/new_jenkins_9.png" >}}
{{< figure src="../../../images/new/new_jenkins_10.png" >}}

### 3. Jenkins – Webhook Step Plugin Installation

Jenkins 대시 보드에서 "Webhook step" 플러그인을 검색하여 설치하십시오. [Jenkins 관리]-> [플러그인 관리]-> [사용 가능]

다음 단계에서 webhook 대신 폴링을 사용하는 apptest.ai Test Stage Code2 소스 코드를 사용하는 경우이 단계를 건너 뛰십시오.

### 4. Jenkins – Pipeline configuration

이 섹션에서는 apptest.ai Test 스테이지를 Jenkins 파이프 라인 항목에 연결하는 방법을 보여줍니다. Jenkins 파이프 라인 항목이 이미 작성되어 있어야합니다.

자세한 내용은 예제 [링크](https://jenkins.io/doc/pipeline/tour/getting-started/)를 참조하십시오.

메인 페이지에서 좌측메뉴의 Configure 버튼을 클릭하십시오.
{{< figure src="../../../images/new/new_jenkins_11.png" >}}
{{< figure src="../../../images/new/new_jenkins_12.png" >}}

파이프 라인 설정 페이지에서 apptest.ai Tes Stage Code를 스크립트 입력란에 추가하십시오.
{{< figure src="../../../images/new/new_jenkins_13.png" >}}

##### [apptest.ai Test Stage Code 1] – Webhook

```

import groovy.json.\*

node {
def apkFile
def accessKey
def projectId
def apiHost
def runTestsetUrl
def testStatusCheckUrl
def osType
def testsetInfoLists
def testResult

  // Add to your Preparation Stage
  stage('Preparation') {
      echo "[ apptest.ai ] Current workspace : ${workspace}"
      userID = 'ci_test@apptest.ai'  // [ Modify ] Your apptest.ai's account ( Login ID )
      accessKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxx'  // [ Modify ] Your apptest.ai account's accesskey (Check out your profile page)
      projectId = 1234  // [ Modify ] The apptest.ai's project ID where the test will be run
      apiHost = "https://api.apptest.ai"
      runTestsetUrl = "${apiHost}/openapi/v2/testset"
      osType='ANDROID'  // [ Modify ] Please enter the os type of the test target. (ANDROID | iOS)

      echo "[ apptest.ai ] Preparation successfully."
  }

  // Apptest.ai Test run Stage
  stage('apptestai Test Run') {
      apkFile="/var/jenkins_home/BBC_News_v5.5.1.2.com.apk"
      // Call apptest.ai's Test run API.
      runTestset = sh(returnStdout: true, script: "curl -X POST -u '${userID}:${accessKey}' -F 'app_file=@\"${apkFile}\"' -F 'data={\"pid\": ${projectId}, \"testset_name\": \"${env.BUILD_TAG}\",\"os_type\": \"${osType}\", \"ci_type\": \"jenkins\"}' ${runTestsetUrl}").toString().trim()
      runTestsetResponse = new JsonSlurperClassic().parseText(runTestset)
      // Test run fail case
      if (runTestsetResponse.result_code != 0) {
          echo "[ apptest.ai ] Failed to run the test. ${runTestset}"
          runTestset = null
          runTestsetResponse = null
          error "FAIL"
      }

      testsetId = runTestsetResponse['data']['testset_id']
      runTestset = null
      runTestsetResponse = null

      echo "[ apptest.ai ] Test run successfully. (Testset ID : ${testsetId})"
  }

  // Apptest.ai Test status check Stage
  stage('apptestai Test Status Check') {
      completeAll = false
      testResultXml = ''

      // Repeat until all tests are complete
      while(!completeAll) {
          // Check the test status for each item in the testsetId lists
          sleep (time: 10, unit: "SECONDS")
          testStatusCheckUrl="${apiHost}/openapi/v2/testset/${testsetId}"
          getTestStatusCheck = sh( returnStdout: true, script: "curl -X GET -u '${userID}:${accessKey}' ${testStatusCheckUrl}").toString().trim()
          getTestStatusCheckResponse = new JsonSlurperClassic().parseText(getTestStatusCheck)
          // Test status check fail case
          if (getTestStatusCheckResponse.result_code != 0) {
              echo "[ apptest.ai ] Failed to get test status. ${getTestStatusCheck}"
              getTestStatusCheck = null
              getTestStatusCheckResponse = null
              error "FAIL"
          }

          testStatus = getTestStatusCheckResponse.data.testset_status.toLowerCase()
          echo "[ apptest.ai ] testset ${testsetId} complete result : ${testStatus}"

          // Get the result data when the test is complete
          if (testStatus == 'complete') {
              testResultUrl = "${apiHost}/openapi/v2/testset/${testsetId}/result"
              getTestResult = sh( returnStdout: true, script: "curl -X GET -u '${userID}:${accessKey}' '${testResultUrl}'").toString().trim()
              getTestResultResponse = new JsonSlurperClassic().parseText(getTestResult)
              // Test result data get fail case
              if (getTestResultResponse.result_code != 0) {
                  echo "[ apptest.ai ] Failed to get test result data. ${getTestResult}"
                  getTestResult = null
                  getTestResultResponse = null
                  error "FAIL"
              }

              // Merge test result xml data
              testResultXml = getTestResultResponse.data.result_xml
              echo "[ apptest.ai ] Test completed. tsid : ${testsetId}"
              getTestResult = null
              getTestResultResponse = null
              completeAll = true
          }
          getTestStatusCheck = null
          getTestStatusCheckResponse = null
      }
      echo "[ apptest.ai ] All tests are completed."
      echo "[ apptest.ai ] Test result data : ${testResultXml}"
  }

  // Write apptest.ai test result data to file Stage
  stage('Write Apptestai Test Result') {
      sh "mkdir -p tmp/"
      // Write File file:"tmp/TESTS-Apptestai.xml", text: testResultXml, encoding: "UTF-8"
      sh "echo -n '${testResultXml}' > tmp/TESTS-Apptestai.xml"
  }

  // JUnit Test Stage
  stage('jUnit Test') {
      junit 'tmp/TESTS-*.xml'
  }

}

```

##### [apptest.ai Test Stage Code 2] – Multi Scenario Test with Polling

시나리오 테스트를 진행하기 위해서는 시나리오 아이디와 소스타입, os type이 반드시 있어야 합니다.

```

import groovy.json.*

node {
    def apkFile
    def accessKey
    def projectId
    def apiHost
    def getScenarioListUrl
    def runTestsetUrl
    def testStatusCheckUrl
    def scenarioQueryString
    def osType
    def scenarioIdLists
    def testsetInfoLists
    def testResult

    // Add to your Preparation Stage
    stage('Preparation') {
        echo "[ apptest.ai ] Current workspace : ${workspace}"
        userID = 'ci_test@apptest.ai'  // [ Modify ] Your apptest.ai's account ( Login ID )
        accessKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxx'  // [ Modify ] Your apptest.ai account's accesskey (Check out your profile page)
        projectId = 1234  // [ Modify ] The apptest.ai's project ID where the test will be run
        apiHost = "https://api.apptest.ai"
        getScenarioListUrl = "${apiHost}/project/${projectId}/scenarios"
        runTestsetUrl = "${apiHost}/openapi/v2/testset"
        scenarioQueryString = ''  // [ Modify ] When getting a list of scenarios, enter the characters you want included in the scenario name.
        osType='ANDROID'  // [ Modify ] Please enter the os type of the test target. (ANDROID | iOS)

        // Get scenario list from project (filter : OS Type , Query String )
        getScenarioList = sh(returnStdout: true, script: "curl -X GET -u '${userID}:${accessKey}' '${getScenarioListUrl}?os_type=${osType}&q=${scenarioQueryString}'").toString().trim()
        getScenarioListResponse = new JsonSlurperClassic().parseText(getScenarioList)
        echo "[ apptest.ai ] getScenarioListResponse : ${getScenarioListResponse}"
        if (getScenarioListResponse.result_code != 0) {
            echo "[ apptest.ai ] Failed to get list of scenarios. ${getScenarioList}"
            getScenarioList = null
            getScenarioListResponse = null
            error "FAIL"
        }
        scenarioIdLists = getScenarioListResponse.data.scenarios
        getScenarioList = null
        getScenarioListResponse = null

        echo "[ apptest.ai ] Preparation successfully."
    }

    // Apptest.ai Test run Stage
    stage('apptestai Test Run') {
        testsetInfoLists = []
        scenarioIdLists.each{
            scenarioId ->
            // Call apptest.ai's Test run API.
            runTestset = sh(returnStdout: true, script: "curl -X POST -u '${userID}:${accessKey}' -F 'data={\"pid\": ${projectId}, \"testset_name\": \"${env.BUILD_TAG}\", \"scenario_id\": ${scenarioId}, \"source_type\": \"file\", \"os_type\": \"${osType}\", \"ci_type\": \"jenkins\"}' ${runTestsetUrl}").toString().trim()
            runTestsetResponse = new JsonSlurperClassic().parseText(runTestset)
            // Test run fail case
            if (runTestsetResponse.result_code != 0) {
                echo "[ apptest.ai ] Failed to run the test. ${runTestset}"
                runTestset = null
                runTestsetResponse = null
                error "FAIL"
            }

            testsetId = runTestsetResponse['data']['testset_id']
            echo "[ apptest.ai ]  TestSet id : ${testsetId} is ran successfully"
            testsetInfo = [:]
            testsetInfo['testset_id'] = testsetId
            testsetInfo['complete'] = false
            testsetInfoLists << testsetInfo
            runTestset = null
            runTestsetResponse = null
        }

        echo "[ apptest.ai ] Test run successfully."
    }

    // Apptest.ai Test status check Stage
    stage('apptestai Test Status Check') {
        completeAll = false
        testResultXml = ''

        // Repeat until all tests are complete
        while(!completeAll) {
            // Check the test status for each item in the testsetId lists
            for (int i = 0; i < testsetInfoLists.size(); i++) {
                waitUntil{
                    sleep (time: 10, unit: "SECONDS")
                    item = testsetInfoLists[i]
                    testsetId = item.testset_id
                    if (item.complete == false) {
                        testStatusCheckUrl="${apiHost}/openapi/v2/testset/${testsetId}"
                        getTestStatusCheck = sh( returnStdout: true, script: "curl -X GET -u '${userID}:${accessKey}' ${testStatusCheckUrl}").toString().trim()
                        getTestStatusCheckResponse = new JsonSlurperClassic().parseText(getTestStatusCheck)
                        // Test status check fail case
                        if (getTestStatusCheckResponse.result_code != 0) {
                            echo "[ apptest.ai ] Failed to get test status. ${getTestStatusCheck}"
                            getTestStatusCheck = null
                            getTestStatusCheckResponse = null
                            error "FAIL"
                        }

                        testStatus = getTestStatusCheckResponse.data.testset_status.toLowerCase()
                        echo "[ apptest.ai ] testset ${testsetId} complete result : ${testStatus}"

                        // Get the result data when the test is complete
                        if (testStatus == 'complete') {
                            testResultUrl = "${apiHost}/openapi/v2/testset/${testsetId}/result"
                            getTestResult = sh( returnStdout: true, script: "curl -X GET -u '${userID}:${accessKey}' '${testResultUrl}?data_type=bare_xml'").toString().trim()
                            getTestResultResponse = new JsonSlurperClassic().parseText(getTestResult)
                            // Test result data get fail case
                            if (getTestResultResponse.result_code != 0) {
                                echo "[ apptest.ai ] Failed to get test result data. ${getTestResult}"
                                getTestResult = null
                                getTestResultResponse = null
                                error "FAIL"
                            }

                            // Test result xml data init value setting
                            if (testResultXml.length() == 0) {
                                testResultJson = new JsonSlurperClassic().parseText(getTestResultResponse.data.result_json)
                                testResultXml = ""
                            }
                            // Merge test result xml data
                            testResultXml = testResultXml + getTestResultResponse.data.result_bare_xml
                            echo "[ apptest.ai ] Test completed. tsid : ${testsetId}"
                            getTestResult = null
                            getTestResultResponse = null
                            item['complete'] = true
                        }
                        getTestStatusCheck = null
                        getTestStatusCheckResponse = null
                    }
                    return true

                }
            }
            // Check running test item
            runningTestItem = testsetInfoLists.findAll { element -> element.complete == false }
            // Marking all tests complete when running test count is zero
            if ( runningTestItem.size() == 0 ) {
                testResultXml = testResultXml + ""
                completeAll = true
            }
        }
        echo "[ apptest.ai ] All tests are completed."
        echo "[ apptest.ai ] Test result data : ${testResultXml}"
    }

    // Write apptest.ai test result data to file Stage
    stage('Write Apptestai Test Result') {
        sh "mkdir -p tmp/"
        // Write File file:"tmp/TESTS-Apptestai.xml", text: testResultXml, encoding: "UTF-8"
        sh "echo -n '${testResultXml}' > tmp/TESTS-Apptestai.xml"
    }

    // JUnit Test Stage
    stage('jUnit Test') {
        junit 'tmp/TESTS-*.xml'
    }
}

```

위 스크립트에서 다음 항목들을 수정하십시오.

- <div style="margin-bottom:10px;">
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">userID </span>
      <span >: apptest.ai의 로그인 아이디</span>
  </div>
- <div style="margin-bottom:10px;">
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">accessKey </span>
      <span >: apptest.ai의 액세스 키</span>
  </div>
- <div style="margin-bottom:10px;">
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">projectId </span>
      <span >: apptest.ai에 생성된 프로젝트의 고유번호 (Project ID)</span>
  </div>

  - apptest.ai 프로젝트 설정에 저장된 Time Limit과 디바이스 정보를 적용하여 테스트를 수행합니다.

  - 샘플 테스트 프로젝트의 설정 변경은 허용되지 않지만 새 프로젝트의 설정은 변경할 수 있습니다.

- <div style="margin-bottom:10px;">
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">osType </span>
      <span >: 테스트 대상의 os type</span>
  </div>
- <div style="margin-bottom:10px;">
      <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">apkFile </span>
      <span >: 테스트 할 대상 앱파일(App Binary File)</span>
  </div>

파이프 라인을 시작하려면 Jenkins에서 "Build Now"를 클릭하십시오.
{{< figure src="../../../images/new/new_jenkins_14.png" >}}

### 5. Test Results

테스트가 완료되면 JUnit XML 결과 형식의 테스트 결과가 Callback URL을 통해 Jenkins에 자동으로 전달됩니다. Jenkins는 반환 된 테스트 결과를 반영합니다.

자세한 테스트결과 분석 정보는 [apptest.ai](https://apptest.ai)를 방문해 확인하실 수 있습니다.

Jenkins에서 테스트 결과보기
{{< figure src="../../../images/new/new_jenkins_15.png" >}}
{{< figure src="../../../images/new/new_jenkins_16.png" >}}
{{< figure src="../../../images/new/new_jenkins_17.png" >}}
{{< figure src="../../../images/new/new_jenkins_18.png" >}}
{{< figure src="../../../images/new/new_jenkins_19.png" >}}
{{< figure src="../../../images/new/new_jenkins_20.png" >}}

apptest.ai 에서 테스트 결과보기
{{< figure src="../../../images/new/new_jenkins_21.png" >}}
{{< figure src="../../../images/new/new_jenkins_22.png" >}}
{{< figure src="../../../images/new/new_jenkins_23.png" >}}
{{< figure src="../../../images/new/new_jenkins_24.png" >}}

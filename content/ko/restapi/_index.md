---
title: "Rest APIs for CI/CD"
linkTitle: "Rest APIs"
weight: 4
description: >

#   What does your user need to understand about your project in order to use it - or potentially contribute to it?
---

## Integrating REST APIs into CI / CD pipelines

CI (Continuous Integration) 도구과 연동하여 apptest.ai를 사용하기위한 API 정보를 제공합니다. 해당 문서에서 제공되는 테스트시작, 테스트 상태 보기, 테스트 결과 조회 API를 통해 apptest.ai를 사용하실 수 있습니다.

### 1. [POST] Run New Testset

새로운 테스트 세트를 생성합니다. (테스트 세트는 단위 테스트들의 집합니다.)<br/>
새로운 테스트기 실행될때 사용되는 일부 옵션들은 프로젝트에 미리 저장해놓은 설정 정보를 따릅니다.

[Request]

```
POST /openapi/v2/testset

Host : https://api.apptest.ai

Authorization : Basic {user_id}:{access_key}
```

Request Body Miltipary Form Data #1

| Key                                                                                                     | Type                                                                                                | Description                                                               | Required |
| :------------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------ | :------- |
| <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">app_file</span> | <span style="background-color: #f9f2f4;color: #c7254e;padding: 5px;border-radius: 5px;">File</span> | <p>테스트할 대상 앱 파일 automated</p> <p>테스트일 경우 필수 값</p></div> | Optional |

Request Body Miltipary Form Data #2

| Key          |          | Type            |                                         | Description                                                                                                                                          | Required |
| :----------- | :------- | :-------------- | :-------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- | :------- |
| pid          |          | Positive Number | -                                       | 프로젝트의 고유번호 <br/> ex) 509                                                                                                                    | Required |
| testset_name |          | String          | Max 100 Characters                      | 테스트 세트의 이름 <br/> ex) "Testset name Example#1"                                                                                                | Required |
| scenario_id  |          | Number          | 0 : automated test<br/> Positive Number | 시나리오 고유번호 ex) 509”                                                                                                                           | optional |
| source_type  |          | String          | file or app_id                          | 시나리오테스트 전용<br/>테스트 대상의 종류 (앱 파일 또는 앱 아이디)<br/>프로젝트의 앱 저장소에 설정된 대상의 종류<br/>시나리오 테스트 진행 시 필수\* | optional |
| os_type      |          | String          | ANDROID or iOS                          | 테스트 대상의 os type<br/> ex) “ANDROID”                                                                                                             | Required |
| time_limit   |          | Positive Number | Min : 5                                 | 테스트 제한시간(분) ex) 5<br/>테스트 시간 제한이 비어 있으면, 프로젝트 설정에 저장된 시간 제한을 따릅니다.                                           | optional |
| use_vo       |          | Boolean         | Default: false                          | Automated test 전용<br/>Whether AT&T Video Optimizer(ARO) is used (true or false )<br/>ex) true                                                      | optional |
| callback     |          | String          | Max 250 Characters                      | 테스트가 완료될 경우 호출될 Callback URL <br/>ex) 'https://127.0.0.1/callback/url/example'                                                           | optional |
| credentials  | login_id | String          | Max 150 Characters                      | 앱을 테스트하는데 사용될 테스트 대상앱의 계정정보(아이디) <br/>(Test credentials info - Login ID) <br/>ex) 'credentials_id'                          | optional |
|              | login_pw | String          | Max 150 Characters                      | 앱을 테스트하는데 사용될 테스트 대상앱의 계정정보(비밀번호) <br/>(Test credentials info - Login PW) <br/>ex) 'credentials_pw'                        | optional |

[ Request Example #1 – Automated Test]

```
curl    --request POST \
        --user {user_id}:{access_key} \
        -F 'app_file=@/path/of/your/app_file' \
        -F 'data={\
            "pid": 509, \
            "testset_name": "Testset Name Example #1", \
            "time_limit": 5, \
            "use_vo": false, \
            "credentials": { \
                "login_id": "credentials_id", \
                "login_pw": "credentials_pw" \
            } \
        }' https://api.apptest.ai/openapi/v2/testset
```

[ Request Example #1 – Scenario Test]

```
curl    --request POST \
        --user {user_id}:{access_key} \
        -F 'data={\
            "pid": 509, \
            "testset_name": "Testset Name Example #1", \
            "os_type": "ANDROID", \
            "scenario_id": 1204, \
            "source_type": "file", \
            "time_limit": 15, \
        }' https://api.apptest.ai/openapi/v2/testset
```

[ Response ]

| key        |     | type            | Description                                         |
| :--------- | :-- | :-------------- | :-------------------------------------------------- |
| test_count |     | Positive Number | 테스트 세트에 실행된 단일 테스트의 갯수 <br/> ex) 3 |
| testset_id |     | Positive Number | 테스트 세트의 고유번호 <br/> ex) 251929             |

[ Response Example ]

```
{
    "data": {
        "test_count": 1,
        "testset_id": 251929
    },
    "result_code": 0,
    "result_msg": "",
    "reason": ""
}
```

[ Error Code ]

| result_code | result_msg                | reason                                                                                                                                                                 |
| :---------- | :------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 4000        | Missing Request Parameter | The required parameter was not found in the request parameters : {{ PARAMETER KEY }}                                                                                   |
| 4030        | App File Analysis Failed  | iOS plistlib Parsing Error : {{ PARSING ERROR MSG }}                                                                                                                   |
| 4031        |                           | Test creation failed : There was a problem pre-processing your app file. \n The IPA file must be a development version that is signed by your development certificate. |
| 4032        |                           | Test creation failed. Invalid app file. \n There was a problem pre-processing your app file.                                                                           |
| 4040        |                           | Android Manifest Parsing Error : {{ PARSING ERROR MSG }}                                                                                                               |
| 4041        |                           | No launchable activity found. (android.intent.category.LAUNCHER)                                                                                                       |
| 4050        | Invalid Request Parameter | File extension not supported. ( Support : ipa, zip, app, apk, xapk, apks )                                                                                             |
| 4051        |                           | The required parameter was not found in the request parameters : (app_file)                                                                                            |
| 4052        |                           | Invalid request parameter : (project_id: {{ PROJECT_ID }})                                                                                                             |
| 4053        |                           | Invalid request parameter : (testset_id: {{ TESTSET_ID }})                                                                                                             |
| 4055        |                           | Device is not compatible with this app : ( APP os type : {{ APP_OS_TYPE }} )                                                                                           |
| 4080        | Invalid Request Parameter | Device info does not exist in prese project data.                                                                                                                      |
| 4081        |                           | Available devices does not exist in preset project data.                                                                                                               |
| 5001        | Run New Testset Error     | Global Exception Error - {{ ERROR_MSG }}                                                                                                                               |
| 6001        | App File Upload Error     | App file upload fail.                                                                                                                                                  |
| 8000        | System Maintenance        | The service is currently under system maintenance.                                                                                                                     |

### 2. [ GET ] Testset Status

테스트 세트의 진행상황 확인

[ Request ]

```
GET /openapi/v2/testset/{testset_id}

Host : https://api.apptest.ai

Authorization : Basic {user_id}:{access_key}
```

Request URL Parameter

| Key        | Type            | Description             | Required |
| :--------- | :-------------- | :---------------------- | :------- |
| testset_id | Positive number | 테스트 세트의 고유 번호 | Required |

[ Request Example ]

```
curl --request GET \
     --user {user_id}:{access_key} \
     https://api.apptest.ai/openapi/v2/testset/55716
```

[ Response ]

Response Body Data Type : JSON

| Key                   |                  | Type            | Description                                     |
| :-------------------- | :--------------- | :-------------- | :---------------------------------------------- |
| testset_status        |                  | String          | 테스트 세트의 진행 상태 ( Complete or Running ) |
| testset_status_detail |                  | JSON            | 테스트의 상태별 갯수                            |
|                       | total_test_cnt   | Positive number | 전체 단일 테스트 갯수                           |
|                       | error_cnt        | Positive number | 에러가 발생된 단일 테스트의 갯수                |
|                       | fail_cnt         | Positive number | 실패한 단일 테스트의 갯수                       |
|                       | initializing_cnt | Positive number | 준비단계의 단일 테스트 갯수                     |
|                       | pass_cnt         | Positive number | 정상종료된 단일 테스트의 갯수                   |
|                       | running_cnt      | Positive number | 테스트가 진행중인 단일테스트의 갯수             |
|                       | stop_cnt         | Positive number | 사용자에 의해 중지된 단일 테스트의 갯수         |
| response_time         |                  | Datetime        | 응답 시간 (timezome : UTC)                      |

[Response Example ]

```
{
    "data": {
        'testset_status': 'Complete',
        'testset_status_detail': {
            'total_test_cnt': 3,
            'error_cnt': 1,
            'fail_cnt': 1,
            'initializing_cnt': 0,
            'running_cnt': 1,
            'stop_cnt': 0,
            'pass_cnt': 0
        },
        'response_time': 'Mon, 25 May 2020 06:40:24 GMT'
    },
    "result_code": 0,
    "result_msg": "",
    "reason": ""
}
```

[ Error Code ]

| result_code | result_msg               | reason                                             |
| :---------- | :----------------------- | :------------------------------------------------- |
| 5002        | Get Testset Status Error | Global Exception Error – {ERROR_MSG}               |
| 8000        | System Maintenance       | The Service is currently under system maintenance. |

### 3. [ GET ] Testset Result

테스트가 완료되었을 경우에만 호출이 가능한 API 입니다. 테스트 세트의 결과 데이터를 조회합니다.

[ Request ]

```
GET /openapi/v2/testset/{testset_id}/result

Host : https://api.apptest.ai

Authorization : Basic {user_id}:{access_key}
```

Request URL Parameter

| Key        | Type            | Description            | Required |
| :--------- | :-------------- | :--------------------- | :------- |
| testset_id | Positive number | 테스트 세트의 고유번호 | Required |

Request Query Parameter

| Key       | Type   | Description                                                                                                                                                  | Required |
| :-------- | :----- | :----------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- |
| data_type | String | Test reuslt data type</br>Default : all (all, xml, bare_xml, json, html)</br>ex) https://api.apptest.ai/openapi/v2/testset/{testset_id}/result?data_type=all | Optional |

[ Request Example ]

```
curl --request GET \
     --user {user_id}:{access_key} \
     https://api.apptest.ai/openapi/v2/testset/55716/result
```

```
curl --request GET \
     --user {user_id}:{access_key} \
     https://api.apptest.ai/openapi/v2/testset/55716/result?data_type=xml
```

[ Response ]

Response Body Data Type : JSON

| Key             | Type    | Description                                                                                                                  |
| :-------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------- |
| complete        | Boolean | 테스트가 진행중인지 종료되었는지에 대한 여부 ( true or false)                                                                |
| result_xml      | Boolean | JUnit형식의 XML 결과 데이터                                                                                                  |
| result_bare_xml | Boolean | Result data in XML format in JUnit format</br>Exclude "?xml" and "testsuites" tag</br>여러테스트를 한번에 실행하는 경우 사용 |
| result_html     | Boolean | HTML형식의 결과 데이터                                                                                                       |
| result_json     | Boolean | JUnit 형식의 Json 결과 데이터                                                                                                |

[ JUnit XML Format ]

| Key        | Type    | Description                               |
| :--------- | :------ | :---------------------------------------- |
| testsuite  | name    | apptest.ai’s 프로젝트 이름                |
| testcase   | name    | 테스트 장치 이름 (Unit Test)              |
|            | time    | Result data in XML format in JUnit format |
| system-out |         | 테스트 결과 링크                          |
| error      | message | 오류가 반견된 테스트 결과 링크            |

[ JUnit XML Format Example ]

```
<?xml version="1.0" encoding="UTF-8"?>
<testsuites name="TestBot Test">
    <testsuite name="{PROJECT NAME}.TestBot">
        <testcase name="{DEVICE NAME}" time="{TEST TIME SECONDS}">
            <system-out>{RESULT PAGE LINK URL}</system-out>
        </testcase>
        <testcase name="{DEVICE NAME}" time="{TEST TIME SECONDS}">
            <error message="{RESULT PAGE LINK URL}" />
        </testcase>
    </testsuite>
</testsuites>
```

[ Error Code ]

| result_code | result_msg                     | reason                                                         |
| :---------- | :----------------------------- | :------------------------------------------------------------- |
| 5003        | Get Testset Result Data Error  | Global Exception Error – {ERROR_MSG}                           |
| 6002        | Get Testset Result Data Failed | Test is running yet. Please request when the test is complete. |
| 6003        | Data Does Not Exist            | Test data does not exist.                                      |
| 8000        | System Maintenance             | The Service is currently under system maintenance.             |

## 4. [ GET ] Scenario ID Lists

시나리오 테스트 실행 시 사용될 시나리오 정보를 조회하기 위한 API 입니다.

[ Request ]

```
GET /openapi/v2/project/{project_id}/scenarios

Host : https://api.apptest.ai

Authorization : Basic {user_id}:{access_key}
```

Request URL Parameter

| Key        | Type            | Description       | Required |
| :--------- | :-------------- | :---------------- | :------- |
| project_id | Positive number | 프로젝트 고유번호 | Required |

Request Query Parameter

| Key     | Type   | Description                                                                                                                                         | Required |
| :------ | :----- | :-------------------------------------------------------------------------------------------------------------------------------------------------- | :------- | -------- |
| os_type | String | 시나리오 등록시 선택한 os type 정보</br>(ANDROID 또는 iOS)</br>ex) https://api.apptest.ai/openapi/v2/project/{project_id}/scenarios?os_type=ANDROID |          | Required |
| q       | String | 시나리오 이름에 해당 문자열이 포함된 목록을 조회</br>ex) https://api.apptest.ai/openapi/v2/project/{project_id}/scenarios?q=login                   | Required |

[ Request Example ]

```
curl --request GET \
     --user {user_id}:{access_key} \
     https://api.apptest.ai/openapi/v2/project/2019/scenarios?os_type=ANDROID&q=login_fail
```

[ Response ]

Response Body Data Type : JSON

| Key           | Type     | Description                |
| :------------ | :------- | :------------------------- |
| scenarios     | Array    | 시나리오 아이디 목록       |
| response_time | Datetime | 응답 시간 (timezome : UTC) |

[ Response Example ]

```
{
    "data": {
        "scenarios": [1204,1206],
        "response_time": "Mon, 25 May 2020 06:40:24 GMT",
    },
    "result_code": 0,
    "result_msg": 0,
    "reason": ""
}
```

[ Error Code ]

| result_code | result_msg                | reason                                             |
| :---------- | :------------------------ | :------------------------------------------------- |
| 2001        | Get Project Data Fail     | Project data does not exist                        |
| 4051        | Invalid request parameter | Invalid request parameter : (project id)           |
| 5005        | Get Scenario List Error   | Global Exception Error - {ERROR_MSG}               |
| 8000        | System Maintenance        | The Service is currently under system maintenance. |

## Appendix. API Result Codes

| result_code | result_msg                     | reason                                                                                                                                                                 |
| :---------- | :----------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 4000        | Missing Request Parameter      | The required parameter was not found in the request parameters : ({PARAMETER KEY})                                                                                     |
| 4030        | App File Analysis Failed       | iOS plistlib Parsing Error : {PARSING ERROR MSG}                                                                                                                       |
| 4031        | App File Analysis Failed       | Test creation failed : There was a problem pre-processing your app file. \n the IPA file must be a development version that is signed by your development certificate. |
| 4032        | App File Analysis Failed       | Test creation failed : Invalid app file. \n There was a problem pre-processing your app file.                                                                          |
| 4040        | App File Analysis Failed       | Android Manifest Parsing Error : {PARSING ERROR MSG}                                                                                                                   |
| 4041        | App File Analysis Failed       | No launchable activity found. (android.intent.category.LAUNCHER)                                                                                                       |
| 4050        | Invalid Request Parameter      | File extension not supported. (support : ipa, zip, app, apk, xapk, apks)                                                                                               |
| 4051        | Invalid Request Parameter      | The required parameter was not found in the request parameters: (app_file)                                                                                             |
| 4052        | Invalid Request Parameter      | Invalid request parameter : (project_id : {PROJECT_ID})                                                                                                                |
| 4053        | Invalid Request Parameter      | Invalid request parameter : (testset_id : {TESTSET_ID})                                                                                                                |
| 4055        | Invalid Request Parameter      | Device is not compatible with this app : (App os type : {APP_OS_TYPE})                                                                                                 |
| 4080        | Invalid Request Parameter      | Device info does not exist in preset project data.                                                                                                                     |
| 4081        | Invalid Request Parameter      | Available devices does not exist in preset project data.                                                                                                               |
| 5001        | Run New Testset Error          | Global Exception Error – {ERROR_MSG}                                                                                                                                   |
| 5002        | Get Testset Status Error       | Global Exception Error – {ERROR_MSG}                                                                                                                                   |
| 5003        | Get Testset Result Data Error  | Global Exception Error – {ERROR_MSG}                                                                                                                                   |
| 6001        | App File Upload Error          | App file upload fail.                                                                                                                                                  |
| 6002        | Get Testset Result Data Failed | Test is running yet. Please request when the test is complete.                                                                                                         |
| 6003        | Data Does Not Exist            | Test data does not exist.                                                                                                                                              |
| 8000        | System Maintenance             | The Service is currently under system maintenance.                                                                                                                     |

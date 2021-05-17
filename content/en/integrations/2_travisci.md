---
title: "2. Travis CI"
date: 2020-05-15
type: docs
weight: 4
description: >
#   A short lead descripton about this content page. It can be **bold** or _italic_ and can be split over multiple paragraphs.
---

## apptest.ai Integration for Travis CI

With just a few lines of script, you can enable apptest.ai along with Travis CI for out testbot to explore your apps and report results for every build process.

To use the apptest.ai service within Travis CI pipeline, you can take the following two steps: 1. Register the apptest.ai Access Key, and 2. add the apptest.ai to your build script (travis.yml)

### Adding apptest.ai access key onto the build environment

- Copy the access key string from the apptest.ai service profile page
{{< figure src="../../images/2_travisci_1.gif" >}}


- Add apptest.ai access key to the build envirronment variable named APPTEST_AI_ACCESS_KEY and add the encryprion to the build script

  ```
  $ travis login
  $ travis encrypt APPTEST_AI_ACCESS_KEY="ID:Access Key"  --add
  ```

  > The format for the access key becomes “apptest.ai ID: apptest.ai Access Key“. For example, if the ID is jean@apptest.ai and the access key is “ab8f3e321d631c84c9b1113”, the following applies:

  ```
  jean@apptest.ai:ab8f3e321d631c84c9b1113
  ```

  After this command, the following “env:” section is added to the existing build script file. This section is replaced by the APPTEST_AI_ACCESS_KEY environment variable declaration at run time.

  ```
  env:
    global:
    - secure: fUnmuuqKmxWjxGHbfTB7QZgSmkB7yvt6KGAn8dCs1j4GneWbSIrEhghRrlOMQCThhMoetou4H+NaWeTXRRa4fpoCGm1U51RbD+J4DrbKWj3MzRupzByEAsaWhmvCaYVgCOlkYgaApcvujIrpw1rUTQVio8+WoYIXgXx3B
  ```

- Now, add a script that executes the following after cloning the CI Tool integration script set by apptest.ai as follows.

  ```
  - git clone https://github.com/apptestai/ci_addon
  - export binary_path=HackerNews.ipa
  - export project_id='825'
  - bash ./ci_addon/step.sh
  ```

  > At least two arguments, binary_path and project_id, are required to invoke the service. binary_path is the path to the binary you want to test, i.e. * .apk files for android and * .ipa files for iOS. project_id is the Project ID for apptest.ai that contains the settings such as test time and device list for testing. The Project ID of apptest.ai can be found on the project information page as shown below.
  > 
  {{< figure src="../../images/2_travisci_2.png" >}}



  If you don’t want to download apptest.ai’s CI integration script every time, include apptest.ai’s ci_addon repository in the Project Repository in advance.

- Once you commit the build script, Travis CI will execute. When running apptest.ai script, the test is executed as shown below and the result is printed on the console.
  {{< figure src="../../images/2_travisci_3.png" >}}


- The test results are stored in test-results/apptest-ai_result.html with detailed HTML report and an XML document in JUnit Test result format in test-results/apptestai/result.xml. You can save this file externally using various deploy functions provided by Travis CI. Below is an example of a release on Github. You can see the apptest.ai test report posted on Github’s release page.
  {{< figure src="../../images/2_travisci_4.gif" >}}


  > If you want to change the location of the folder to store results, declare and pass test_result_path environment variable. If you want to execute the next build without waiting for the test to finish, declare waiting_for_test_results environment variable and pass FALSE.


  Finally, here’s the .travis.yml example. First, for Android, in the “env” section, you can see that there are two secure environment variables for storing apptest.ai access key and two secure environment variables for Github access key to deploy.

  {{< gist genie-park afb71bfb6981edbc109eb5b8511007de >}}



  For an iOS case, iOS needs two more registered environment variables. This is an environment variable for code sign which is required when creating binary on iOS. Below we used a Fastlane match for the code sign. Fastlane match encrypts the certificate file and provisioning file required for code signing on Github, so two more access tokens and decryption keys are added to access a private GitHub repository where the certificate is stored compared to Android.


  {{< gist genie-park 8c3756623d48b472e177bc473c5c9338 >}}

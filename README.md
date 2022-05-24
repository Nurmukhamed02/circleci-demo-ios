# Testing IOS mobile Application in [CircleCi](https://circleci.com/)

## This is a flexible CircleCi pipeline for building, testing, and releasing our IOS mobile application. In order to perform these tasks, we use _Fastlane._

## Using Fastlane
### [Fastlane](https://fastlane.tools/)  is a set of tools for automating the build and deploy process of mobile apps. We encourage the use of Fastlane on CircleCI as it simplifies the setup and automation of the build, test and deploy process. Additionally, it allows parity between local and CircleCI builds.

## Setting up Fastlane for use on CircleCI
When using Fastlane in your CircleCI project, we recommend adding the following to beginning of your Fastfile:
```
...
platform :ios do
  before_all do
    setup_circle_ci
  end
  ...
end
```
### In this file we can choose on what kind of platform we want to test App and include the different kind of tests which we going to test when Pipeline will run. And in this Fastfile we have _test_ and _release_ lane
### This pipeline is reusable because here we are using parameterized build:
```
9      parameters:
10         fastlane:
11            default: "test"
12            description: Choose the relevant lane option for running fastlane
13            type: enum
14            enum: ["test", "release"]
```
### The _enum_ parameter may be a list of any values. Use the enum parameter type when you want to enforce that the value must be one from a specific set of string values. Which means we can choose only _test_ or _release_ lane. If default parameter was specified it will take that one. But under the _workflow_ if you specified _parameter lane_ it will overwrite the _default_ one:
```
workflows:
  build-test-release:
    jobs:
      - fastlane:
          lane: "release"
```
## Branching strategy
### After each event for all branches ( commit, pull request, etc) the CircleCi Pipeline will be executed. In order to prevent this, we can use a _Branching Strategy_. A Branching Strategy will restrict which means we can specify to run the pipeline in the specific branch(s). To implement this we have to specify under the workflow:
```
workflows:
  build-test-release:
    jobs:
      - fastlane:
          lane: "release"
          filters:
            branches:
              only: DEV  #run only on the DEV branch
```
## App store connect
### To set up Fastlane to automatically upload iOS binaries to App Store Connect and/or TestFlight, a few steps need to be followed to allow Fastlane access to your App Store Connect account

### It is recommended to generate and use an App Store Connect API key. To create an API Key, follow the steps outlined in the [Apple DeveloperDocumentation](https://developer.apple.com/documentation/appstoreconnectapi/creating_api_keys_for_app_store_connect_api). Once you have the resulting .p8 file, make a note of the Issuer ID and Key ID which can be found on the [App Store Connect API Keys page](https://appstoreconnect.apple.com/login?targetUrl=%2Faccess%2Fapi&authResult=FAILED).

### Next, a few environment variables need to be set. In your CircleCI project, navigate to _Build Settings -> Environment Variables_ and set the following:
- `APP_STORE_CONNECT_API_KEY_ISSUER_ID`  to the Issuer ID
     - Example: `6053b7fe-68a8-4acb-89be-165aa6465141`
- `APP_STORE_CONNECT_API_KEY_KEY_ID` to your Key ID
     - Example: `D383SF739`
- `APP_STORE_CONNECT_API_KEY_KEY` to the contents of your .p8 file
     - Example: `-----BEGIN PRIVATE KEY-----\nMIGTAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBHknlhdlYdLu\n-----END PRIVATE KEY-----`

### Finally, Fastlane requires some information from us in order to know which Apple ID to use and which app identifier we are targeting. These can be set in the fastlane/Appfile as follows:
```
1   # fastlane/Appfile
2   apple_id "ci@yourcompany.com"
3   app_identifier "com.example.HelloWorld"

```
## Deploying to TestFlight
### [TestFlight](https://developer.apple.com/testflight/) is a tool from Apple which allows beta testers to install and test an application before it is officially released in the app store.
### The example below shows how Fastlane can be configured to automatically build, sign and upload an iOS binary. Pilot has lots of customisation options to help deliver apps to TestFlight, so it is highly recommended to check out the [pilot documentation](https://docs.fastlane.tools/actions/pilot/) for further information.
```
1    # fastlane/Fastfile
2    default_platform :ios
3
4    platform :ios do
5      before_all do
6        setup_circle_ci
7      end
8
9      desc "Upload to Testflight"
10      lane :upload_testflight do
11        # Get the version number from the project and check against
12        # the latest build already available on TestFlight, then
13        # increase the build number by 1. If no build is available
14        # for that version, then start at 1
15        increment_build_number(
16          build_number: latest_testflight_build_number(
17            initial_build_number: 1,
18            version: get_version_number(xcodeproj: "HelloWorld.xcodeproj")
19          ) + 1,
20        )
21        # Set up Distribution code signing and build the app
22        match(type: "appstore")
23        gym(scheme: "HelloWorld")
24        # Upload the binary to TestFlight and automatically publish
25        # to the configured beta testing group
26        app_store_connect_api_key
27        pilot(
28          distribute_external: true,
29          notify_external_testers: true,
30          groups: ["HelloWorld Beta Testers"],
31          changelog: "This is another new build from CircleCI!"
32        )
33      end
34    end

```
  


## For more information how to implement this testing pipeline visit the [official Documentation of Circleci](https://circleci.com/docs/2.0/testing-ios/)





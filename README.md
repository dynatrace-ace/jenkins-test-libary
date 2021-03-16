# Overview

Jenkins shared library for test executions

## Usage

The directive `@Library('jenkinstest@main')` is added to the top of a Jenkinsfile script to load the latest version of the Jenkins Test Library. Then within the various pipeline stages, Test library functions are called with the required and optional parameters.

Library versions are listed below:  

| Library Version | Comment |
| --------------- | ------- |
| v1.0 | Initial Release |


*It is recommended to specify the library version in the Jenkinsfile to ensure pipeline stability. For example `@Library('jenkinstest@v1.0')`*

## Library functions:

**1. Execute Jmeter Test**
  * Supporting function that allows to run jmeter tests from Jenkins. This function assumes we run on a Jenkins Agent that has JMeter installed 
  * See [Jmeter README](JMETER.md) for usage details

Once you have everything configured use it in your Jenkins Pipeline like this

```groovy

// Import Dynatrace library -- the v1.0 indicates the tag/branch to use.
@Library("jenkins-test@v1.0")

// Initialize the class with the event methods
def jmeter = new com.dynatrace.ace.Jmeter()

// this is called with a script step
pipeline {
  agent jmeter
  stages {
    stage('Run performance test') {
      steps {
        container('jmeter') {
          script {
            def status = jmeter.executeJmeterTest ( 
                scriptName: "jmeter/simplenodeservice_load.jmx",
                resultsDir: "perfCheck_${env.APP_NAME}_staging_${BUILD_NUMBER}",
                serverUrl: "simplenodeservice.staging", 
                serverPort: 80,
                checkPath: '/health',
                vuCount: env.VU.toInteger(),
                loopCount: env.LOOPCOUNT.toInteger(),
                LTN: "perfCheck_${env.APP_NAME}_${BUILD_NUMBER}",
                funcValidation: false,
                avgRtValidation: 4000
            )
            if (status != 0) {
                currentBuild.result = 'FAILED'
                error "Performance test in staging failed."
            }
          }
        }
      }
    }
  }
}

```

# Setup

## Prerequisites

**#1 - Jenkins server**  

You may have your own, but if not one option is to run Jenkins as [Docker container](https://github.com/jenkinsci/docker/blob/master/README.md).  This command will start it up and prompt for setting up initial user and default plugins.
```
docker run -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

## Install and configure the Dynatrace Jenkins Library

1. Login to Jenkins 
1. Navigate to Manage Jenkins > Configure System

    ![](./images/config-menu.png)

1. Find the **Global Pipeline Libraries** section, click add new and fill in as shown below

    * Select **Git** as the type
    * Project repository = https://github.com/dynatrace-ace/jenkins-test-library.git

    ![](./images/config-lib.png)

# Support

If you’d like help with this pipe, or you have an issue or feature request, let us know. The repo is maintained by Dynatrace. You can contact us directly at ace@dynatrace.com.

If you’re reporting an issue, please include:

* the version of the pipe
* relevant logs and error messages
* steps to reproduce

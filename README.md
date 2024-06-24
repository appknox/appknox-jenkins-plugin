# Appknox Jenkins Plugin

The Appknox Jenkins Plugin allows you to perform Appknox security scan on your mobile application binary. The APK/IPA built from your CI pipeline will be uploaded to Appknox platform which performs static scan and the build will be errored according to the chosen risk threshold.

## How to use it?

### Step 1: Get your Appknox access token

Sign up on [Appknox](https://appknox.com).

Generate a personal access token from <a href="https://secure.appknox.com/settings/developersettings" target="_blank">Developer Settings</a>

### Step 2: Configure the Jenkins Plugin

In your Jenkinsfile Add this after building your Application stage.

```
stages {
        stage('Appknox Scan') {
            steps {
                script {
                    step([
                        $class: 'AppknoxPlugin',
                        accessToken: params.APPKNOX_ACCESS_TOKEN,
                        filePath: params.FILE_PATH,
                        riskThreshold: params.RISK_THRESHOLD.toUpperCase()
                    ])
                }
            }
        }
    }
    
```

## Inputs

| Key                     | Value                        |
|-------------------------|------------------------------|
| `appknox_access_token`  | Personal access token secret |
| `file_path`             | File path to the mobile application binary to be uploaded |
| `risk_threshold`        | Risk threshold value for which the CI should fail. <br><br>Accepted values: `CRITICAL, HIGH, MEDIUM & LOW` <br><br>Default: `LOW` |

---

Example:
```
pipeline {
    agent any
    parameters {
        string(name: 'APPKNOX_ACCESS_TOKEN', defaultValue: '', description: 'Appknox Access Token')
        choice(name: 'RISK_THRESHOLD', choices: ['LOW', 'MEDIUM', 'HIGH', 'CRITICAL'], description: 'Risk Threshold')
    }
    stages {
        stage('Build App') {
            steps {
                sh './gradlew assembleDebug' 
                
                // Capture the path to the generated APK
                script {
                    def apkPath = sh(returnStdout: true, script: 'find . -name "*.apk"').trim()
                    // Assuming there's only one APK, adjust this logic if needed
                    if (apkPath) {
                        // Set FILE_PATH parameter for Appknox scan stage
                        params.FILE_PATH = apkPath
                    } else {
                        error "Failed to find APK file in the build output."
                    }
                }
            }
        }
        stage('Appknox Scan') {
            steps {
                script {
                    // Perform Appknox scan using AppknoxPlugin
                    step([
                        $class: 'AppknoxPlugin',
                        accessToken: params.APPKNOX_ACCESS_TOKEN,
                        filePath: params.FILE_PATH,
                        riskThreshold: params.RISK_THRESHOLD.toUpperCase()
                    ])
                }
            }
        }
    }
}

```
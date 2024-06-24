//Example pipeline script
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
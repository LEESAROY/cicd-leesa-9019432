pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'cicd-jenkins'
        FUNCTION_APP_NAME = 'node-leesa-9019432'
    }

    stages {
        stage('Setup') {
            steps {
                script {
                    echo 'Installing dependencies...'
                    bat '''
                        @echo off
                        SETLOCAL

                        REM Install Chocolatey if not already installed
                        choco -v
                        if %ERRORLEVEL% NEQ 0 (
                            powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))"
                        )

                        REM Install zip
                        choco install zip -y
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building the application...'
                    bat 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                    bat 'npm test'
                }
            }
        }

        stage('Deploy') {
            environment {
                AZURE_CLIENT_ID = credentials('AZURE_CLIENT_ID')
                AZURE_CLIENT_SECRET = credentials('AZURE_CLIENT_SECRET')
                AZURE_TENANT_ID = credentials('AZURE_TENANT_ID')
            }
            steps {
                script {
                    echo 'Deploying to Azure...'
                    bat '''
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        zip -r function.zip .
                        az functionapp deployment source config-zip --resource-group %RESOURCE_GROUP% --name %FUNCTION_APP_NAME% --src function.zip
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo 'Cleaning up...'
                    bat 'del function.zip'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

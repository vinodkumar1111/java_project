pipeline {
    agent any

    tools {
        maven 'maven-3'
        jdk 'jdk-17'
    }

    environment {
        // This name must match the name in the Jenkins system configuration
        SONAR_SERVER = 'SonarQube'
    }

    stages {
        stage('Initialize') {
            steps {
                echo 'Starting the build pipeline'
                sh 'mvn --version'
            }
        }

        stage('Parallel Build & Check') {
            parallel {
                stage('Build & Test') {
                    steps {
                        echo 'Compiling and Testing...'
                        // Clean, remove old files, compile, run unit tests, and build jar
                        sh 'mvn clean package'
                    }
                    post {
                        always {
                            // Save test results for Jenkins UI
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }

                stage('Checkstyle') {
                    steps {
                        echo 'Checking code formatting...'
                        // Run Checkstyle plugin
                        sh 'mvn checkstyle:checkstyle'
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Scanning with SonarQube...'

                    // Use withSonarQubeEnv to set up necessary environment variables
                    withSonarQubeEnv('SonarQube') {
                        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                            sh "mvn sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // Pause pipeline and wait for SonarQube to give a verdict (Pass/Fail)
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate(abortPipeline: true)
                }
            }
        }
    }

    post {
        failure {
            slackSend (channel: '#builds', message: "Build failed!")
        }
    }
}

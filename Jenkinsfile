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
        stage ('SonarQube Analysis')
        {
            steps{
                script {
                    echo 'Scannning with SonarQube..'
                    //This wrapper injects the token and url automatically
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage ('Quality Gate'){
            steps{
                //pause pipeline and wait for SonarQube to give a verdict (Pass/Fail)
                timeout(time: 5, unit: 'MINUTES'){
                    waitForQualityGate()
                }
            }
        }
    }
}
    
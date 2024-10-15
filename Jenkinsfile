pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool name: 'sonar_scanner'// type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/Yaswanth8991/secretsanta-generator.git'
            }
        }

        // stage('Parallel Stages') {
        //     parallel {
        stage('Code Compile') {
            steps {
                        sh "mvn clean compile"
                }
            }

        stage('Unit Tests') {
            steps {
                        sh "mvn test"
                }
            }
        //     }
        // }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=Santa \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Santa
                    '''
                }
            }
        }

        stage('Quality Gate Analysis') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
            }
        }

        stage('Build Stage') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'settings-file', 
                    maven: 'maven', 
                    traceability: true
                ) {
                    sh "mvn deploy"
                }
            }
        }
                
        stage('Docker Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t santa123 ."
                    }
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker tag santa123 yaswanth98/santa123:latest"
                        sh "docker push yaswanth98/santa123:latest"
                    }
                }
            }
        }
    }
}

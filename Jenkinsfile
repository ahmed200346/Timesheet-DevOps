pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Compose') {
            steps {
                sh 'docker compose config --quiet'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvn = tool 'Default Maven'
                    withSonarQubeEnv('SQ1') {
                        sh "${mvn}/bin/mvn -f backend/pom.xml clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=TestSonarQube -Dsonar.projectName='TestSonarQube' -Dsonar.host.url=http://172.17.159.52:9000"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build and Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_TOKEN'
                )]) {
                    sh '''
                        set -eu
                        set +x
                        export DOCKERHUB_USER="$DH_USER"
                        export IMAGE_TAG="$BUILD_NUMBER"

                        docker compose build backend frontend
                        printf '%s' "$DH_TOKEN" | docker login --username "$DH_USER" --password-stdin
                        docker compose push backend frontend
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout >/dev/null 2>&1 || true'
        }
    }
}
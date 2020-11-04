pipeline {

    agent any

    tools {
        maven 'M3'
    }

    stages {

        stage('maven test') {
            steps {
                sh 'mvn test'
            }
        }
        post {
            success {
                slackSend(color: 'good', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] has been updated and pulled from Github.")
                }
            failure {
                slackSend(color: 'danger', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] failed in unit testing.")
                }
            }
        }

        stage('maven build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('sonarqube quality check') {
            steps {
                withSonarQubeEnv("SonarCloud")
                {
                sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.branch.name=\"master\""
                sleep(10)
                }
            }
        }

        stage('sonar quality gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
            post {
                success {
                    slackSend(color: 'good', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] has passed the SonarQube quality gate.")
                }
                failure {
                    slackSend(color: 'danger', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] failed the Sonar quality gate.")
                }
            }
        }

        stage('docker build and tag') {
            steps {
                sh "docker build -t dougliu/testweb:${currentBuild.number} ."
                sh "docker tag dougliu/testweb:${currentBuild.number} dougliu/testweb:latest"
            }
        }

        stage('docker push to dockerhub') {
            steps {
                withDockerRegistry([credentialsId: 'DockerCred', url: '']) {
                    sh "docker push dougliu/testweb:${currentBuild.number}"
                    sh "docker push dougliu/testweb:latest"
                }
            }
            post {
                success {
                    slackSend(color: 'good', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] new image is built and pushed to Dockerhub.")
                    }
                failure {
                    slackSend(color: 'danger', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] failed push the new image to dockerhub.")
                    }
                }
            }

        stage('deploy to remote K8S cluster') {
            steps {
                withKubeConfig([credentialsId: 'k8sCred', serverUrl: '${KUBEURL}']) {
                    sh 'kubectl apply -f deploy.yaml'
                    sh 'kubectl rollout restart deployment/sample-app'
                 }
             }
            post {
                success {
                    slackSend(color: 'good', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] deployment to K8S is successful")
                }
                failure {
                    slackSend(color: 'danger', message: "Maven project '${JOB_NAME}' [${GIT_BRANCH}] failed to deployer to k8s")
                }
            }
        }
    }
}
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
        }

        stage('deploy to remote K8S cluster') {
            steps {
                withKubeConfig([credentialsId: 'k8sCred', serverUrl: '${KUBEURL}']) {
                    sh 'kubectl apply -f deploy.yaml'
                    sh 'kubectl rollout restart deployment/sample-app'
                 }
             }
        }
    }
}
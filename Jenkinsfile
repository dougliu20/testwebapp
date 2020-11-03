pipeline {

    agent any

    stages {
        stage('poull from github') {
            steps {
                git url: "https://github.com/dougliu20/testwebapp", branch: "master"
            }
        }

        stage('sonarqube quality check') {
            tools {
                maven 'M3'
            }
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
                withSonarQubeEnv("SonarCloud")
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('docker build and tag') {
            agent any
            steps {
                sh "docker build -t dougliu/testweb:${currentBuild.number} ."
                sh "docker tag dougliu/testweb:${currentBuild.number} dougliu/testweb:latest"
            }
        }

        stage('docker push to dockerhub') {
            agent any
            steps {
                withDockerRegistry([credentialsId: 'DockerCred', url: '']) {
                    sh "docker push dougliu/testweb:${currentBuild.number}"
                    sh "docker push dougliu/testweb:latest"
                }
            }
        }
        
        stage('deploy to remote K8S cluster') {
            agent any
            steps {
                withKubeConfig([credentialsId: 'k8sCred', serverUrl: '${KUBEURL}']) {
                    sh 'kubectl apply -f deploy.yaml'
                    sh 'kubectl rollout restart deployment/sample-app'
                 }
             }
        }
    }
}
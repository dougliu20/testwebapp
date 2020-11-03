pipeline {

    agent none

    stages {
        stage('build') {
            agent any
            tools {
                maven 'M3'
            }
            steps {
                sh 'mvn package'
            }
        }
        stage('Sonarqube') {
            agent any
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
        stage('docker build') {
            agent any
            steps {
                sh "docker build -t dougliu/testweb:${currentBuild.number} ."
                sh "docker tag dougliu/testweb:${currentBuild.number} alxl/sample-spring-boot:latest"
            }
        }
        stage('docker push') {
            agent any
            steps {
                withDockerRegistry([credentialsId: 'DockerCred', url: 'https://registry.hub.docker.com']) {
                    sh 'docker push dougliu/testweb:${currentBuild.number}'
                    sh 'docker push dougliu/testweb:latest'
                }
            }
        }
        stage('app deploy') {
            
            agent any

            steps {
                    sh 'kubectl apply -f deploy.yaml'
                    sh 'kubectl rollout restart deployment/sample-app'
                }
            // agent {
            //     docker { image '' }
            // }
            // steps {
            //     withKubeConfig([credentialsId: 'kubectl-creds', serverUrl: '']) {
            //         sh 'kubectl apply -f deploy.yaml'
            //         sh 'kubectl rollout restart deployment/sample-app'
            //     }
            // }
        }
    }
}
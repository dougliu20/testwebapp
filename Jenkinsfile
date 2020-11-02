pipeline {
    agent any

    tools {
        maven 'M3'
    }

    stages {
        stage('Pull from Git') {
            steps {
                git url: "https://github.com/dougliu20/testwebapp", branch: "master"
            }
        }

        stage('Test using Maven') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }    
        stage('Check with SonarCloud') {
            steps {
                //Link to Sonar Cloud: https://sonarcloud.io
                withSonarQubeEnv("SonarCloud")
                    {
                    sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.branch.name=\"master\""
                    sleep(10)
                    }
            }
        }
        stage("Sonar Quality Gate ") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Upload Docker Image') {
            steps {
                script {
                   docker.withTool('docker') {
                        repoId = "dougliu/testweb:${currentBuild.number}"
                        image = docker.build(repoId)
                        docker.withRegistry("https://registry.hub.docker.com", "DockerCred") {
                        image.push()
                        }
                    }
                }
            }
        }
        stage('Rollout') {
            steps {
                    sh 'kubectl apply -f deploy.yaml'
                    sh 'kubectl rollout restart deployment/sample-app'
            }
        }
                                                                                                                    }
}
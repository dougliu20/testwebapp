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
                sh 'mvn test -Dcheckstyle.skip=true'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true -Dcheckstyle.skip=true'
            }
        }    
        stage('Check with SonarCloud') {
            steps {
                withSonarQubeEnv("SonarCloud")
                    {
                    sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar"
                    }
            }
        }
        stage('Upload Docker Image') {
            steps {
                script {
                   docker.withTool('docker') {
                        repoId = "dougliu/testweb"
                        image = docker.build(repoId)
                        docker.withRegistry("https://registry.hub.docker.com", "dockercred") {
                        image.push()
                        }
                    }
                }
            }
        }
        stage('Rollout') {
            steps {
                    sh 'kubectl apply -f deploy.yaml'
                    //sh 'kubectl rollout restart deployment/sample-deployment'
            }
        }
                                                                                                                    }
}
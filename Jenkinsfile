pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "M3"
    }

    stages {
        stage('GitHub Pull') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/dougliu20/testwebapp'
            }
        }
        stage('Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn clean package"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage('Sonar Cloud Test') {
            steps {
                withSonarQubeEnv("<Name of SonarQube server>")
                {
                    sh "mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar"
                }
            }
        }
    }
}
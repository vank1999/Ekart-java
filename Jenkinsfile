pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }

    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'git-cred',
                url: 'https://github.com/vank1999/Ekart-java.git'
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('trivy fs scan') {
            steps {
                sh "trivy fs ."
            }
        }

        stage('owasp fs-scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
               sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=EKART '''
                }
            }
        }
        
        stage('build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        } 

        stage('deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings-xml') {
                sh "mvn deploy -DskipTests=true"
            }
        }
      }

        stage('build & tag docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "docker build -t shopping-cart:dev -f docker/Dockerfile ."
                     sh "docker tag shopping-cart:dev vank1999/shopping-cart:dev"
                  }
                }
            }
        }

        stage('push docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "docker push vank1999/shopping-cart:dev"
                  }
                }
            }
        }

        stage('deploy application') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "docker run -d --name ekart -p 8070:8070 vank1999/shopping-cart:dev"
                  }
                }
            }
        }

    }
}

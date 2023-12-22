pipeline {
    agent any

    // tools {
    //     jdk 'jdk11'
    //     maven 'maven3'
    // }
    tools {
        jdk 'jdk17'
        maven 'maven3.9.6'
        dockerTool 'docker'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                script {
                    git branch: 'main', changelog: false, credentialsId: '721cf0ff-87ad-4210-9c9b-d25745f2fe0c', poll: false, url: 'https://github.com/alaomichael/Ekart.git'
                }
            }
        }

        stage('Compile') {
            steps {
                script {
                    echo "Current directory: ${pwd()}"
                    sh 'mvn clean compile -DskipTests=true'
                }
            }
        }

        stage('OWASP Scan') {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage('Sonarqube') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                            -Dsonar.projectName=Shopping-Cart \
                            -Dsonar.java.binaries=. \
                            -Dsonar.projectKey=Shopping-Cart
                        '''
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'mvn clean package -DskipTests=true'
                }
            }
        }

        stage('Build Docker Image') {
        steps {
            script {
                      docker.build('shopping-cart:latest', '-f docker/Dockerfile .')
                    }
                }
         }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: '14a89b32-1d5b-4ccf-994a-eb26529fd2b8', usernameVariable: 'HUB_USER', passwordVariable: 'HUB_TOKEN')]) {                      
                    sh '''
                        docker login -u $HUB_USER -p $HUB_TOKEN 
                        docker image tag shopping-cart:latest $HUB_USER/shopping-cart:latest
                        docker image push $HUB_USER/shopping-cart:latest
                    '''
                }
            }
        }
        
        stage('Deploy Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: '14a89b32-1d5b-4ccf-994a-eb26529fd2b8', usernameVariable: 'HUB_USER', passwordVariable: 'HUB_TOKEN')]) {                      
                    sh '''
                        docker login -u $HUB_USER -p $HUB_TOKEN 
                        docker run -d --name shop-shop -p 8070:8070 $HUB_USER/shopping-cart:latest
                        '''
                }
            }
        }
      
          }
}

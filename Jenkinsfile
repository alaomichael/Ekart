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
                    git branch: 'dev', changelog: false, credentialsId: '721cf0ff-87ad-4210-9c9b-d25745f2fe0c', poll: false, url: 'https://github.com/alaomichael/Ekart.git'
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
                            -Dsonar.projectName=Shopping-Cart-Dev \
                            -Dsonar.java.binaries=. \
                            -Dsonar.projectKey=Shopping-Cart-Dev
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
                      docker.build('shopping-cart-dev:latest', '-f docker/Dockerfile .')
                    }
                }
         }
        
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: '14a89b32-1d5b-4ccf-994a-eb26529fd2b8', usernameVariable: 'HUB_USER', passwordVariable: 'HUB_TOKEN')]) {                      
                    sh '''
                        docker login -u $HUB_USER -p $HUB_TOKEN 
                        docker image tag shopping-cart-dev:latest $HUB_USER/shopping-cart-dev:latest
                        docker image push $HUB_USER/shopping-cart-dev:latest
                    '''
                }
            }
        }
        
        // stage('Deploy Docker Image') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: '14a89b32-1d5b-4ccf-994a-eb26529fd2b8', usernameVariable: 'HUB_USER', passwordVariable: 'HUB_TOKEN')]) {                      
        //             sh '''
        //                 docker login -u $HUB_USER -p $HUB_TOKEN 
        //                 docker run -d --name shop_dev-shop_dev -p 8060:8060 $HUB_USER/shopping-cart-dev:latest
        //                 '''
        //         }
        //     }
        // }
      
          }
}

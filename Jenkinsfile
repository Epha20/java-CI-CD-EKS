pipeline {
    agent {
        kubernetes {
            label 'jenkins-default-worker'
        } 
    }
    environment {
        DOCKER_REGISTRY = 'harbor.mycompany.com'
        IMAGE_NAME = 'harbor.mycompany.com/group/java-app'
        VERSION = "${BUILD_NUMBER}"      
        VAULT_ADDR = 'http://vault:8200'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'checkingout to GitLab'
                git 'https://gitlab.com/java-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Clean and Install') {
            steps {
                container('maven') {
                    withSonarQubeEnv('SonarQube') {
                        script {
                            sh 'mvn -ntp -B clean install package org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:${VERSION} .'
            }
        }

        stage('Trivy Scan') {
            steps {
                container('trivy') { 
                    sh 'trivy image $DOCKER_REGISTRY/$IMAGE_NAME:${VERSION}'
                }
            }
        }

        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login $DOCKER_REGISTRY -u $USERNAME --password-stdin'
                    sh 'docker push $DOCKER_REGISTRY/$IMAGE_NAME:${VERSION}'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([string(credentialsId: 'vault-token', variable: 'VAULT_TOKEN')]) {
                    sh '''
                      export VAULT_TOKEN=$VAULT_TOKEN
                      DB_PASSWORD=$(vault kv get -field=password secret/db)

                      sed "s|{{DB_PASSWORD}}|$DB_PASSWORD|g" deployment/deployment.yaml > deployment/deployment-pass.yaml
                      kubectl apply -f deployment/deployment-pass.yaml
                      kubectl apply -f deployment/service.yaml
                    '''
                }
            }
        }
    }
}

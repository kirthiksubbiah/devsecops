@Library('shared-lib') _



pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    stages {
        stage("Git Checkout") {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jkbarathkumar/devsecops'
            }
        }

        
        

        stage("Compile") {
            steps {
                sh "mvn clean compile"
            }
        }

        stage("Test Cases") {
            steps {
                sh "mvn test"
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectName=devsecops \
                        -Dsonar.projectKey=devsecops \
                        -Dsonar.java.binaries=.
                    '''
                }
            }
        }

        // 🔁 Replaced local Build stage with shared library call
        stage("Build") {
            steps {
                mavenBuild('clean install')
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t barathkumar29/devsecops-java:v1 .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push barathkumar29/devsecops-java:v1
                    '''
                }
            }
        }

        stage("Kubernetes Deploy") { 
            steps {
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}

pipeline {
    agent any 
    
    tools{
        jdk 'jdk'
        maven 'maven'
    }
    
    environment {
        //SCANNER_HOME=tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jkbarathkumar/devsecops'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    //sh ''' mvn sonar:sonar -Dsonar.url=http://localhost:9000/ -Dsonar.login=sqa_a5cbb592d57dd8f885425eac5cd5fdcfc240cb40 -Dsonar.projectName=devsecops \
                    //-Dsonar.java.binaries=. \
                    //-Dsonar.projectKey=devsecops '''
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectName=devsecops \
                        -Dsonar.projectKey=devsecops \
                        -Dsonar.java.binaries=.
                        '''
    
                }
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
         stage("Build"){
            steps{
                sh " mvn clean install"
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
        
        stage("TRIVY"){
            steps{
                sh " trivy image barathkumar29/devsecops-java:v1"
            }
        }
        
        stage("Kubernetes Deploy"){
            steps{
                sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
            }
        }
    }
}

pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node16' 
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Hema-atyam/Netflix-DevSecOps.git'
            }
        }
        
        stage('sonar analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=netflix \
                    -Dsonar.projectKey=netflix'''
                }
            }
        }
        
        
        stage('installing dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage('dependency check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        
        stage('docker image build and push') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                    sh "docker build --build-arg TMDB_V3_API_KEY=<api_key> -t netflix ."
                    sh "docker tag netflix hemaatyam/netflix:${BUILD_NUMBER}"
                    sh "docker push hemaatyam/netflix:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('image scan') {
            steps {
                sh "trivy image hemaatyam/netflix:${BUILD_NUMBER} > trivyimage.txt"
            }
        }
        
        stage('deploy to container') {
            steps {
                sh "docker run -d -p 8081:80 hemaatyam/netflix:${BUILD_NUMBER}"
            }
        }
    }
}

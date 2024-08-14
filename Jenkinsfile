pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Glenita21/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://192.168.152.128:9000 -Dsonar.login=squ_6a477b4986595af09b17879bcf44a036aa103ddd -Dsonar.projectName=Ekart \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=Ekart '''
            }
        }
        
        stage('OWASP SCAN') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '/dependency-check-report.xml'

            }
        }
        
        stage('Build Application') {
            steps {
                sh "mvn clean install -DskipTests=true"
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                script { 
                    withDockerRegistry(credentialsId: 'e4a612dd-4dc6-4739-bdae-188e0ce5a419', toolName: 'docker') {
                              sh "docker build -t shopping:latest -f docker/Dockerfile ."
                              sh "docker tag shopping:latest glenita/shopping:latest"
                              sh "docker push glenita/shopping:latest"
                    }
                }
    
            }
        
        }
        
        stage('Trigger CD Pipeline') {
            steps {
                build job: "CD-PIPELINE" , wait:true
            }
        }
    }
}

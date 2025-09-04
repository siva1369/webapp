pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment{
        SCANNER_HOME= tool 'sonar'
    }


    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/siva1369/webapp.git'
            }
        }

         stage('compilation') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('tesing') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Code Coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''
                     ${SCANNER_HOME}/bin/sonar-scanner \
                     -Dsonar.projectKey=webapps \
                     -Dsonar.projectName=webapps \
                     -Dsonar.sources=. \
                     -Dsonar.java.binaries=. 
                      '''
                }
            }
        }
        
        stage('OWASP Scan') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'owasp'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                sh '''
                    curl -v -u mounika:mounika \
                    --upload-file target/secretsanta-0.0.1-SNAPSHOT.jar \
                    http://13.203.222.204:8081/repository/webapps/com/notthebest/secretsanta/0.0.1-SNAPSHOT/secretsanta-0.0.1-SNAPSHOT.jar
                '''
            }
        }
        
        stage('docker Build') {
            steps {
                 sh 'docker build -t webapps .'
                 sh 'docker tag webapps siva1369/webapps:latest'
            }
        }


        stage('Trivy image scan') {
            steps {
                 sh 'trivy image siva1369/webapps:latest'
            }
        }
        
         stage('docker push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker') {
                          sh "docker push siva1369/webapps:latest"
                    }
                }
            }
        }

       
    }
}

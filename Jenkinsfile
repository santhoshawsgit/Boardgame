pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/santhoshawsgit/Boardgame.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan . --format HTML --out dependency-check-report --noupdate',
                    odcInstallation: 'dependency-check'
                )
            }
        }

        stage('Publish Report') {
            steps {
                publishHTML(target: [
                    reportDir: 'dependency-check-report',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Report',
                    keepAll: true,
                    alwaysLinkToLastBuild: true
                ])
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'dependency-check-report/**'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                      -Dsonar.projectKey=Boardgame \
                      -Dsonar.projectName=Boardgame \
                      -Dsonar.java.binaries=.
                    '''
                }
            }
        }

        stage('Sonar Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Deploy') {
              steps {
                    withMaven(
                    globalMavenSettingsConfig: 'golbalsettings',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }
        
        
            stage('Docker image create') {
            steps {
                sh 'docker build -t santhoshawsdocker/board:latest .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-fs-report.html santhoshawsdocker/board:latest'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS')]) {
                    sh '''
                        docker login -u $USER -p $PASS
                        docker push santhoshawsdocker/board:latest
                    '''
                }
            }
        }
        
        
                    stage('Docker running') {
            steps {
                sh '''docker rm -f boardcontainer || true
                        docker run -itd --name boardcontainer -p 8082:8080 santhoshawsdocker/board:latest
                        docker ps
                        '''
            }
        }
    }
}

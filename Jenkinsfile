pipeline {
    agent any

    tools {
        maven 'Maven3'        // Your Jenkins Maven tool name
        nodejs 'NodeJS'       // Optional if using Node scripts locally
        snyk 'SnykLatest'     // Your configured Snyk installation
    }

    environment {
        // Optional: set Node/npm environment if needed
        PATH = "${tool 'NodeJS'}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile Stage') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Testing Stage') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                dir('web') {
                    script {
                        // Run Snyk test and save JSON
                        sh 'snyk test --json --severity-threshold=high > snyk_report.json || true'

                        // Convert JSON report to HTML
                        sh 'snyk-to-html -i snyk_report.json -o snyk_report.html'

                        // Archive the HTML report
                        archiveArtifacts artifacts: 'web/snyk_report.html', fingerprint: true
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=jenkins-integration \
                          -Dsonar.projectName="Jenkins Integration" \
                          -Dsonar.sources=src/main/java \
                          -Dsonar.tests=src/test/java \
                          -Dsonar.java.binaries=target/classes
                    ''' // Replace with your SonarQube scanner step
               }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Checking Quality Gate...'
            }
        }

        stage('Package Stage') {
            steps {
                echo 'Packaging application...'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Check Snyk report for vulnerabilities.'
        }
    }
}

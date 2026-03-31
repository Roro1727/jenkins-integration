pipeline {
    agent any

    environment {
        MAVEN_HOME = tool 'maven_3_5_0'
        PATH = "${MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {

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

        stage('OWASP Dependency Check') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    dependencyCheck(
                        additionalArguments: '--scan ./ --format XML --format HTML --out ./dependency-check-report --prettyPrint -n',
                        odcInstallation: 'OWASP-DC'
                    )
                    dependencyCheckPublisher(
                        pattern: 'dependency-check-report/dependency-check-report.xml'
                    )
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
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package Stage') {
            steps {
                sh 'mvn package'
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo '✅ All stages passed successfully!'
        }
        unstable {
            echo '⚠️ Pipeline completed with warnings.'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above.'
        }
    }

}

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
        withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_KEY')]) {
            dependencyCheck(
                additionalArguments: """
                    --scan ./
                    --format XML
                    --format HTML
                    --out ./dependency-check-report
                    --nvdApiKey ${NVD_KEY}
                    --prettyPrint
                """,
                odcInstallation: 'OWASP-DC'
                )
            }
            post {
                always {
                    dependencyCheckPublisher(
                        pattern: 'dependency-check-report/dependency-check-report.xml'
                    )
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
}
}

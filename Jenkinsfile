pipeline {

    agent any

    environment {

        JFROG_URL = "http://20.204.145.235:8082/artifactory/maven-local"
    }

    stages {

        stage('Clone Code') {

            steps {

                git branch: 'master',
                url: 'https://github.com/khwazabani/demo1.git'
            }
        }

        stage('Build Application') {

            steps {

                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Verify Artifact') {

            steps {

                sh '''
                echo "Current Workspace"
                pwd

                echo "Workspace Files"
                ls -la

                echo "Searching for Target Directories"
                find . -type d -name target

                echo "Searching for JAR Files"
                find . -name "*.jar"
                '''
            }
        }

        stage('Upload Artifact to JFrog') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'jfrog-jenkins-cred',
                    usernameVariable: 'JF_USER',
                    passwordVariable: 'JF_PASS'
                )]) {

                    sh '''
                    JAR_FILE=$(find . -name "*.jar" | head -1)

                    echo "Uploading Artifact: $JAR_FILE"

                    curl -u $JF_USER:$JF_PASS \
                    -T $JAR_FILE \
                    "$JFROG_URL/demo-app-${BUILD_NUMBER}.jar"
                    '''
                }
            }
        }

        stage('Build Docker Image') {

            steps {

                sh 'docker --version'

                sh '''
                docker build -t springboot-app:${BUILD_NUMBER} .
                '''

                sh 'docker images'
            }
        }

        stage('Run Docker Container') {

            steps {

                sh '''
                docker rm -f springboot-container || true

                docker run -d \
                --name springboot-container \
                -p 8085:8080 \
                springboot-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Running Container') {

            steps {

                sh 'docker ps'

                sh '''
                echo "Application Logs"
                docker logs springboot-container
                '''
            }
        }
    }

    post {

        success {

            echo 'Pipeline executed successfully'
        }

        failure {

            echo 'Pipeline failed'
        }

        always {

            cleanWs()
        }
    }
}

    
       

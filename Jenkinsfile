pipeline {

    agent {
        docker {
            image 'maven:3.9.6-eclipse-temurin-17'
        }
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

                sh 'mvn clean package'
            }
        }

        stage('List Artifact') {

            steps {

                sh 'pwd'
                sh 'ls -lh target'
                sh 'find target -name "*.jar"'
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
                    curl -u $JF_USER:$JF_PASS \
                    -T target/*.jar \
                    "http://20.204.145.235:8082/artifactory/maven-local/demo-app.jar"
                    '''
                }
            }
        }

     stage('Build Docker Image') {

    agent any

    steps {

        sh 'docker --version'

        sh '''
        docker build -t springboot-app:${BUILD_NUMBER} .
        '''

        sh 'docker images'
    }
}
    }
post{
        success {

            echo 'Artifact uploaded successfully to JFrog'
        }

        failure {

            echo 'Pipeline failed'
        }

        always {

            cleanWs()
        }

    }

}

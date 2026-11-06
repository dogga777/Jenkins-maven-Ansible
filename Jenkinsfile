
pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Source code checkout completed'
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                echo 'Building Spring Boot application'
                bat 'mvn -B clean package'
            }
        }

        stage('Publish Test Results') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'BUILD AND TESTING SUCCESSFUL!'
        }

        failure {
            echo 'BUILD FAILED - CHECK CONSOLE OUTPUT'
        }
    }
}

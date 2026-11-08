
pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'maven3'
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

        stage('Deploy with Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ansible-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {
                    bat '''
                        C:\\Windows\\System32\\OpenSSH\\scp.exe -i "%SSH_KEY%" -P 2222 -o StrictHostKeyChecking=no -o UserKnownHostsFile=NUL target\\demo-1.0.0.jar %SSH_USER%@127.0.0.1:/tmp/demo-1.0.0.jar

                        C:\\Windows\\System32\\OpenSSH\\ssh.exe -i "%SSH_KEY%" -p 2222 -o StrictHostKeyChecking=no -o UserKnownHostsFile=NUL %SSH_USER%@127.0.0.1 "cd /home/vagrant/project/ansible && ansible-playbook -i inventory.ini deploy.yml --extra-vars jar_source=/tmp/demo-1.0.0.jar"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'BUILD AND DEPLOYMENT SUCCESSFUL!'
        }

        failure {
            echo 'BUILD OR DEPLOYMENT FAILED - CHECK CONSOLE OUTPUT'
        }
    }
}

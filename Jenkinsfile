
pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'maven3'
    }

    stages {

        // Stage 1: Checkout source code from GitHub
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        // Stage 2: Build and test the application
        stage('Build and Test') {
            steps {
                echo 'Building Spring Boot application'
                bat 'mvn -B clean package'
            }
        }

        // Stage 3: Publish test results
        stage('Publish Test Results') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }

        // Stage 4: Archive the generated JAR file
        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

        // Stage 5: Deploy application using Ansible
        stage('Deploy with Ansible') {
            steps {
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ubuntu-ansible-ssh',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {

                    bat '''
                        echo Fixing SSH private key permissions...

                        icacls "%SSH_KEY%" /inheritance:r
                        icacls "%SSH_KEY%" /remove "BUILTIN\\Users" "NT AUTHORITY\\Authenticated Users" "Everyone"
                        icacls "%SSH_KEY%" /grant:r "SYSTEM:F"

                        echo Copying application JAR to Ubuntu...

                        C:\\Windows\\System32\\OpenSSH\\scp.exe -i "%SSH_KEY%" -P 2222 -o StrictHostKeyChecking=no -o UserKnownHostsFile=NUL target\\demo-1.0.0.jar %SSH_USER%@127.0.0.1:/tmp/demo-1.0.0.jar

                        
                        if errorlevel 1 exit /b 1

                        echo Verifying application...

                        C:\\Windows\\System32\\OpenSSH\\ssh.exe -i "%SSH_KEY%" -p 2222 -o StrictHostKeyChecking=no -o UserKnownHostsFile=NUL %SSH_USER%@127.0.0.1 "for i in $(seq 1 30); do if curl -fsS http://localhost:8081 >/dev/null; then echo Application is responding; exit 0; fi; sleep 2; done; echo Application failed to start; sudo systemctl status springboot --no-pager; sudo journalctl -u springboot -n 30 --no-pager; exit 1"

                        if errorlevel 1 exit /b 1

                        echo Application deployed and verified successfully!

                    '''
                }
            }
        }
    }

    // Final pipeline status
    post {
        success {
            echo 'BUILD AND DEPLOYMENT SUCCESSFUL!'
        }

        failure {
            echo 'BUILD OR DEPLOYMENT FAILED - CHECK CONSOLE OUTPUT'
        }
    }
}

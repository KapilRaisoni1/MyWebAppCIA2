pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('Docker')
    }
    stages {
        stage('Clone') {
            steps {
                git credentialsId: 'GIT', branch: 'main', url: 'https://github.com/KapilRaisoni1/MyWebAppCIA2.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build --no-cache -t kapil7919/mywebapp:12 .'
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker', passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh '''
                        echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push kapil7919/mywebapp:12
                    '''
                }
            }
        }
        stage('Deploy on AWS') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@43.204.116.192 "
                        docker stop mywebapp || true && docker rm mywebapp || true
                        docker run -d --name mywebapp -p 80:80 kapil7919/mywebapp:12
                        "
                    '''
                }
            }
        }
    }
    post {
        success {
            emailext (
                subject: "Jenkins Build Success: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Good news! Build ${env.BUILD_URL} was successful.",
                to: "kapilwithtech@gmail.com"
            )
        }
        failure {
            emailext (
                subject: "Jenkins Build Failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Build failed. Check console output at ${env.BUILD_URL}",
                to: "kapilwithtech@gmail.com"
            )
        }
    }
}

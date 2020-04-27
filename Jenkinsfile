pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker imgae') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("marind/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILDNUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsID: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERNAME' -v ssh -o StrictHostKeyChecking=n0 $USERNAME@prod_ip \"docker pull marind/train-schedule:latest"
                        try {
                            sh "sshpass -p '$USERNAME' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERNAME' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERNAME' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d marind/train-schecdule:latest\""
                    }
                }
            }
        }                
    }
}

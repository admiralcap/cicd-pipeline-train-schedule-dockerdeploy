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
        stage('Build Docker Image') {
            when {
                branch 'ooga'
            }
            steps {
                script {
                    app = docker.build("bossdock/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'ooga'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToDev') {
            when {
                branch 'ooga'
            }
            steps {
                input 'Deploy to Dev?'
                milestone(1)
                script {
                    sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerDev \"docker pull bossdock/train-schedule:${env.BUILD_NUMBER}\""
                    try {
                        sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerDev \"docker stop train-schedule\""
                        sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerDev \"docker rm train-schedule\""
                    } catch (err) {
                        echo: 'caught error: $err'
                    }
                    sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerDev \"docker run --restart always --name train-schedule -p 8080:8080 -d bossdock/train-schedule:${env.BUILD_NUMBER}\""
                }
            }
        }
        //stage('DeployToProduction') {
        //    when {
        //        branch 'ooga'
        //    }
        //    steps {
        //        input 'Deploy to Production?'
        //        milestone(1)
        //        script {
        //            sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerProd \"docker pull bossdock/train-schedule:${env.BUILD_NUMBER}\""
        //            try {
        //                sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerProd \"docker stop train-schedule\""
        //                sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerProd \"docker rm train-schedule\""
        //            } catch (err) {
        //                echo: 'caught error: $err'
        //            }
        //            sh "ssh -o StrictHostKeyChecking=no cloud_user@$DockerProd \"docker run --restart always --name train-schedule -p 8080:8080 -d bossdock/train-schedule:${env.BUILD_NUMBER}\""
        //        }
        //    }
        //}
    }
}

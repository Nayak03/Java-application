pipeline {
    agent any

    stages {
        
        stage('Maven Build') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build') {
            steps {
                sh "docker build . -t kammana/java-application:${commit_id()}"
            }
        }
        stage('Docker Push') {
            steps {
                withCredentials([string(credentialsId: 'docker-hub', variable: 'hubPwd')]) {
                    sh "docker login -u kammana -p ${hubPwd}"
                    sh "docker push kammana/java-application:${commit_id()}"
                }
            }
        }
        stage('Docker Deploy') {
            steps {
                sshagent(['docker-host']) {
                    sh "ssh -o StrictHostKeyChecking=no  ec2-user@172.31.36.37 docker rm -f java-application"
                    sh "ssh  ec2-user@172.31.36.37 docker run -d -p 8080:8080 --name hiring kammana/java-application:${commit_id()}"
                }
            }
        }

    }
}

def commit_id(){
    id = sh returnStdout: true, script: 'git rev-parse HEAD'
    return id
}

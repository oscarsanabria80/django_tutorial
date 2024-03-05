pipeline {
    agent none
    stages {
        stage ('Testear django') { 
            agent { 
                docker { image 'python:3'
                args '-u root:root'
                }
            }
            stages {
                stage('Clone') {
                    steps {
                        git branch:'master',url:'https://github.com/oscarsanabria80/django_tutorial.git'
                    }
                }
                stage('Install') {
                    steps {
                        sh 'pip install -r requirements.txt'
                    }
                }
                stage('Test') {
                    steps {
                        sh 'python3 manage.py test'
                    }
                } 
            }
        }
        stage('Upload img') {
            agent any
            stages {
                stage('Build and push') {
                    steps {
                        script {
                            withDockerRegistry([credentialsId: 'DOCKER_HUB', url: '']) {
                            def dockerImage = docker.build("oscarsanabria80/django_tutorial1:latest")
                            dockerImage.push()
                            }
                        }
                    }
                }
                stage('Remove image') {
                    steps {
                        script {
                            sh "docker rmi oscarsanabria80/django_tutorial1:latest"
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent any
            steps {
                script {
                    sshagent(credentials: ['SSH_VPS']) {
                        sh "ssh -o StrictHostKeyChecking=no oscar@oscarsanabria.blog docker-compose -f /home/oscar/django_tutorial/docker-compose.yaml down"
                        sh "ssh -o StrictHostKeyChecking=no oscar@oscarsanabria.blog docker rmi oscarsanabria80/django_tutorial1:latest"
                        sh "ssh -o StrictHostKeyChecking=no oscar@oscarsanabria.blog docker pull oscarsanabria80/django_tutorial1:latest"
                        sh "ssh -o StrictHostKeyChecking=no oscar@oscarsanabria.blog wget https://raw.githubusercontent.com/oscarsanabria80/django_tutorial/master/docker-compose.yaml -O docker-compose.yaml"
                        sh "ssh -o StrictHostKeyChecking=no oscar@oscarsanabria.blog docker-compose up -d --force-recreate"
                    }
                }
            }
        }
    }
    post {
        always {
            mail to: 'oscarponcedeleonsanabria@gmail.com',
            subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
            body: "${env.BUILD_URL} has result ${currentBuild.result}"
        }
    }
}

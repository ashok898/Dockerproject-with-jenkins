# Dockerproject-with-jenkins
i have deployed this web application using docker container with Jenkins pipeline  and used Sonar qube for code quality , nexus for artifact store and maven for build and some other tools aswell


pipeline {
    agent {
        node {
            label 'dockerslave'
        }
    }

    tools {
        maven 'mymaven'
    }

    stages {

        stage('CleanWS') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'project',
                    credentialsId: 'gitcredentials',
                    url: 'https://github.com/ashok898/dockerwebapp.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'cp -r target docker-app'
            }
        }

        stage('Artifact') {
            steps {
                nexusArtifactUploader artifacts: [[
                        artifactId: 'vprofile',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war'
                ]],
                credentialsId: 'nexusid',
                groupId: 'com.visualpathit',
                nexusUrl: '172.177.239.59:8087',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'mydockerrepo',
                version: 'v2'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t ashok898/mydockerproject:appimage Docker-app'
                sh 'docker build -t ashok898/mydockerproject:dbimage Docker-db'
            }
        }
        
        stage('push code') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhubdetails', url: 'https://index.docker.io/v1/') {
                    sh 'docker push ashok898/mydockerproject:appimage'
                    sh 'docker push ashok898/mydockerproject:dbimage'
                    
                }
                
            }
            
        }
        
        stage('deploy') {
            steps {
                sh 'docker stack deploy --compose-file=docker-compose.yml myappstack --with-registry-auth'
                
            }
            
        }
        
    }
}



<img width="1917" height="929" alt="image" src="https://github.com/user-attachments/assets/c554bd48-08d0-4851-a7e8-785c976f958f" />

<img width="1894" height="826" alt="image" src="https://github.com/user-attachments/assets/2e743330-bba9-4feb-b09e-181f2801022c" />

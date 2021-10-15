pipeline {
   agent none 

   

    stages {
        stage('Compile et tests') {
            agent {
                docker {
                    image 'maven:3.8.3-jdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            } 
            steps {
                echo 'Unit test et packaging'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                dir('application/target') {
                    stash includes: '*.jar', name: 'app'
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    archiveArtifacts artifacts: 'application/target/*.jar', followSymlinks: false
                }
                failure {
                    mail bcc: '', body: 'http://localhost:7979/job/multi-branche/job/dev', cc: '', from: '', replyTo: '', subject: 'Packaging failed', to: 'david.thibau@gmail.com'
                }
            }
        }
        stage('Analyse qualité et test intégration') {
            parallel {
                stage('Tests d integration') {
                    agent {
                docker {
                    image 'maven:3.8.3-jdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            } 
                    steps {
                        echo 'Tests d integration'
                        sh 'mvn -Dmaven.test.failure.ignore=true clean integration-test'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     agent {
                docker {
                    image 'maven:3.8.3-jdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            } 
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dmaven.test.failure.ignore=true clean verify sonar:sonar'
                     }
                    
                }
            }
            
        }
        stage('Push to DockerHub') {
            agent any
            steps {
                unstash 'app'
                sh 'mv application/src/main/docker/Dockerfile ./Dockerfile'
                script {
                    def dockerImage = docker.build('dthibau/multi-module', '.')
                    docker.withRegistry('https://registry.hub.docker.com', 'dthibau_dockerhub') {
                        dockerImage.push 'latest'
                    }
                }
            }
        }   

        stage('Déploiement intégration') {
            agent any
            input {
                message 'Vers quel data-center voulez vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['Paris', 'Lille', 'Orléans'], name: 'DATACENTER'
                }
            }
            steps {
                echo "Déploiement intégration"
                unstash 'app'
                sh "cp *.jar /home/dthibau/Formations/Jenkins/MyWork/Serveur/${DATACENTER}.jar"
                
            }
        }
     }
    
}


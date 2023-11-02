pipeline {
   agent none 

    tools {
        jdk 'JDK17'
    }
    stages {
        stage('Compile et tests') {
            agent any 
            steps {
                echo 'Unit test et packaging'
                sh './mvnw -Dmaven.test.failure.ignore=true clean package'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', followSymlinks: false
                    dir('application/target') {
                        stash includes: '*.jar', name: 'app'
                    }
                }
                failure {
                    mail bcc: '', body: 'Please review', cc: '', from: '', replyTo: '', subject: 'Build failed', to: 'david.thibau@sparks.com'
                }
            }
        }
        stage('Analyse qualité et test intégration') {
            parallel {
                stage('Vulnérabilités') {
                    agent any 
                    steps {
                        echo 'Tests de vulnérabilités'
                        sh './mvnw -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     agent any
                     steps {
                        echo 'Analyse sonar'
                        sh './mvnw -Dmaven.test.failure.ignore=true clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement DATACENTER') {
            agent none
            input {
                message 'Vers quel data center voulez vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['PARIS', 'NANCY', 'STRASBOURG'], name: 'DATACENTER'
                }
            }

            steps {
                echo "Déploiement intégration vers $DATACENTER"
                unstash 'app'
                sh "cp *.jar /home/dthibau/Formations/Jenkins/MyWork/Serveurs/${DATACENTER}.jar"
            }
        }
       

     }
    
}


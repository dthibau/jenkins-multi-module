@Library('DavidLib@dev') _


def integrationUrl
def dataCenters

pipeline {
   agent none 

  
    stages {
        stage('Compile et tests') {
            agent {
                docker {
                    image 'openjdk:17-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                } 
            }
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
                    tools {
                        jdk 'JDK17'
                    }
                    steps {
                        echo 'Tests de vulnérabilités'
                        sh './mvnw -DskipTests verify'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     agent any
                     tools {
                        jdk 'JDK17'
                     }
                     steps {
                        echo 'Analyse sonar'
                        sh './mvnw -Dmaven.test.failure.ignore=true clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
        stage ('Release') {
            agent any

            steps {
                createTarGz(
                    sourceDir: 'library/src/main/java',
                    extensions: ['java'],
                    outputDir: '.',
                    outputFile: 'lib-src-dist'
                )
                sh "cp lib-src-dist.tar.gz /home/dthibau/Formations/Jenkins/MyWork"
            }
        }
        stage('Read Deployment Info') {
            agent any

            steps {
                echo "Read Deployment Info"
                script {
                    def jsonData = readJSON file: './deployment.json'
                    echo "jsonData ${jsonData}"
                    integrationUrl = jsonData.integrationURL
                    dataCenters = jsonData.dataCenters
                    echo "integration ${integrationUrl}"
                    echo "dataCenters ${dataCenters}"
                }
                
            }
        }

        stage('Déploiement vers DATACENTER') {
            agent none

            steps {
                input message: "Voulez vous déployer vers $dataCenters", ok: 'Déployer'
               echo "Deploying ..."
            }
        }

        stage('Déploiement intégration') {
            agent any
          
            steps {
                unstash 'app'
                script {
                    for (dataCenter in dataCenters) {
                        sh "cp *.jar ${integrationUrl}/${dataCenter}.jar"
                    }
                }
            }
        }
       
     }
    
}







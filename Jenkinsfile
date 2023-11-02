def integrationUrl
def dataCenters

pipeline {
   agent none 
    environment {
        DATACENTER =''
    }
  
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







pipeline {
   agent none 

    tools {
        maven 'MAVEN3'
    }


    stages {
        stage('Compile et tests') {
            agent any
            steps {
                echo 'Unit test et packaging'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            } 
            post {
                always {
                    // One or more steps need to be included within each condition's block.
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    // One or more steps need to be included within each condition's block.
                    archiveArtifacts artifacts: '**/application/target/*.jar', followSymlinks: false
                    dir {'application/target'} {
                        stash name: 'application', includes: '*.jar'
                    }
                }
                unsuccessful {
                    // One or more steps need to be included within each condition's block.
                    mail bcc: '', body: 'Pipeline en erreur', cc: '', from: 'jenkins@plbformation.com', replyTo: '', subject: 'Error !', to: 'david.thibau@gmail.com'
                }
            }
             
        }
/*        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any 
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
                            sh 'mvn verify -Dnvd.api.key=$NVD_API_KEY -DskipTests'
                        }
                    }
                    post {
                        success {
                            // One or more steps need to be included within each condition's block.
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'application/target', reportFiles: 'dependency-check-report.html', reportName: 'Analyse de dépendances OWASP', reportTitles: '', useWrapperFileDirectly: true])                        
                        }
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any 
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
  */          
        stage('Déploiement intégration') {
            input {
                message 'Vers quel datacenter voulez-vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['Paris', 'Lille', 'Lyon'], name: 'DATACENTER'
                }
            }

            steps {
                echo "Déploiement intégration $DATACENTER"
                unstash 'application'
                sh 'cp *.jar /home/dthibau/Formations/Jenkins/MyWork/Serveurs/${DATACENTER}.jar'
                
            }
        }

     }
    
}


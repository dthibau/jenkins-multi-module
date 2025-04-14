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
                }
                unsuccessful {
                    // One or more steps need to be included within each condition's block.
                    mail bcc: '', body: 'Pipeline en erreur', cc: '', from: 'jenkins@plbformation.com', replyTo: '', subject: 'Error !', to: 'david.thibau@gmail.com'
                }
            }
             
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any 
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
                            sh 'mvn verify -Dnvd.api.key=$NVD_API_KEY -DskipTests'
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
            
        stage('Déploiement intégration') {

            steps {
                echo "Déploiement intégration"
                
            }
        }

     }
    
}


pipeline {
   agent none 

   tools {
       maven 'MAVEN3'
       jdk 'JDK8'
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
                    agent any
                    steps {
                        echo 'Tests d integration'
                        sh 'mvn -Dmaven.test.failure.ignore=true clean integration-test'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     agent any
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dmaven.test.failure.ignore=true clean test'
                        script {
                            def scannerHome = tool 'SONAR_SCANNER4';
                            withSonarQubeEnv('SONAR_DOCKER') { // If you have configured more than one global server connection, you can specify its name
                                sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar.properties"
                            }
                        }
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
            agent any

            steps {
                echo "Déploiement intégration"
                
            }
        }

     }
    
}


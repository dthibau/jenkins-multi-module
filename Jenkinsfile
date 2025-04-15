def datacenters = []
def integrationURL = ''


pipeline {
   agent none


    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
        timeout(time: 1, unit: 'HOURS')
    }
    triggers {
        pollSCM 'H/2 * * * *';
    }



    stages {
        stage('Compile et tests') {
            agent {
                docker {
                args '-v $HOME/.m2:/root/.m2'
                image 'openjdk:17-alpine'
                }
            }

            steps {
                echo 'Unit test et packaging'
                sh './mvnw -Dmaven.test.failure.ignore=true clean package'
            } 
            post {
                always {
                    // One or more steps need to be included within each condition's block.
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    // One or more steps need to be included within each condition's block.
                    archiveArtifacts artifacts: '**/application/target/*.jar', followSymlinks: false
                    dir ('application/target') {
                        stash name: 'application', includes: '*.jar'
                    }
                }
                unsuccessful {
                    // One or more steps need to be included within each condition's block.
                    mail bcc: '', body: 'Pipeline en erreur', cc: '', from: 'jenkins@plbformation.com', replyTo: '', subject: 'Error !', to: 'david.thibau@gmail.com'
                }
            }
             
        }
        stage('Déploiement vers Artifactory') {
            agent any
            environment {
                ARTIFACTORY = credentials('ARTIFACTORY_USER')
            }
            steps {
                echo 'Déploiement vers Artifactory'
                sh 'env'
                sh './mvnw -s settings.xml -Dartifactory.user=admin -Dartifactory.password=Admin123 deploy -DskipTests'
            }
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any 
                    tools {
                        maven 'MAVEN3'
                    }
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
                    tools {
                        maven 'MAVEN3'
                    }
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                        script {
                            checkSonarQualityGate()
                        }
                     }
                    
                }
            }
            
        }
        stage('Push to Dockerhub') {
            agent any
            steps {
                
                unstash 'application'
                script {
                    def dockerImage = docker.build("dthibau/multi-module", ".")
                    docker.withRegistry('https://registry.hub.docker.com', 'dthibau_docker') {
                        dockerImage.push "${env.BRANCH_NAME}"
                    }
                }
                cleanWs()
            }
        }

        stage('Reading configuration') {
            agent any
            steps {
                script {
                    echo "Reading configuration"
                    def props = readJSON file: 'deployment.json'
                    dataCenters = props['dataCenters']
                    integrationUrl = props['integrationURL']
                }
            }
                
        }
        stage('Validation déploiement') {
            agent none
            steps {
                input message: "Voulez vous déployer vers $dataCenters", ok: 'Déployer'
                echo "Deploying ..."
            }
                
        }
  

        stage('Déploiement intégration') {
            agent any
            steps {
                echo "Déploiement intégration"
                unstash 'application'
                script {
                    for (datacenter in datacenters) {  
                        sh "cp *.jar $integrationURL/${datacenter}.jar"
                    }
                }
                
            }
        }

     }
    
}

def checkSonarQualityGate(){
    // Get properties from report file to call SonarQube 
    def sonarReportProps = readProperties  file: 'target/sonar/report-task.txt'
    def sonarServerUrl = sonarReportProps['serverUrl']
    def ceTaskUrl = sonarReportProps['ceTaskUrl']
    def ceTask

    // Get task informations to get the status
    timeout(time: 4, unit: 'MINUTES') {
        waitUntil(initialRecurrencePeriod: 1000)  {
            withCredentials ([string(credentialsId: 'SONAR_TOKEN', variable : 'token')]) {
                def response = sh(script: "curl -u ${token}: ${ceTaskUrl}", returnStdout: true).trim()
                ceTask = readJSON text: response
            }

            echo ceTask.toString()
              return "SUCCESS".equals(ceTask['task']['status'])
        }
    }

    // Get project analysis informations to check the status
    def ceTaskAnalysisId = ceTask['task']['analysisId']
    def qualitygate

    withCredentials ([string(credentialsId: 'SONAR_TOKEN', variable : 'token')]) {
        def response = sh(script: "curl -u ${token}: ${sonarServerUrl}/api/qualitygates/project_status?analysisId=${ceTaskAnalysisId}", returnStdout: true).trim()
        qualitygate =  readJSON text: response
    }

    echo qualitygate.toString()
    if ("ERROR".equals(qualitygate['projectStatus']['status'])) {
        unstable 'Quality Gate failed'
    }
}
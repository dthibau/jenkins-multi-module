@Library('DavidLib@dev') _


def integrationUrl
def dataCenters

pipeline {
   agent none 

  
    stages {
        stage('Check Ansible') {
            agent {
                kubernetes {
                    yamlFile 'kubernetesPod.yml'
                }
            }
            steps {
                container(name: 'ansible') {
                    sh 'ansible --version'
                }
            }
        }
        stage('Compile et tests') {
            // jdk17-agent a été configuré par l'adminisrateur
            // Il hérite de défaut et ajoute un container openjdk:17-alpine
            agent {
                kubernetes {
                    inheritFrom 'jdk17-agent'
                }
            }
            steps {
                container(name: 'openjdk-17') {
                  echo 'Unit test et packaging'
                  sh 'env | grep JAVA'
                  sh 'javac -version'
                  sh './mvnw -Dmaven.test.failure.ignore=true clean package'
                }
                
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
                    agent {
                        kubernetes {
                            yamlFile 'kubernetesPod.yml'
                        }
                    }
                    steps {
                        container(name: 'jdk') {
                            echo 'Tests de vulnérabilités'
                            sh 'ls -al /root/.m2'
                            sh './mvnw -DskipTests -Dmaven.repo.local=/root/.m2/repository verify'
                            sh 'ls -al /root/.m2'

                        }
                        
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent none
                     steps {
                        echo 'Analyse sonar'
                    //    sh './mvnw -Dmaven.test.failure.ignore=true clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
        stage('Push to DockerHub') {
            agent {
                kubernetes {
                    yamlFile 'kubernetesPod.yml'
                }
            }
            steps {
                container(name: 'dind') {
                    unstash 'app'
                    script {
                        def dockerImage = docker.build('dthibau/multi-module', '.')
                        docker.withRegistry('https://registry.hub.docker.com', 'dthibau_docker') {
                            dockerImage.push "$BRANCH_NAME"
                        }
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
            }
            post {
                success {
                    archiveArtifacts artifacts: 'lib-src-dist.tar.gz', followSymlinks: false
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
                        sh "mv *.jar ${dataCenter}.jar"
                        archiveArtifacts artifacts: "$dateCenter.jar", followSymlinks: false
                    }
                }
            }
        }
       
     }
    
}







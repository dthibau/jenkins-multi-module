pipeline {
   agent none 
    tools {
        jdk 'JDK17'
    }
    stages {
        stage('Compile et tests') {
            agent {
                label 'jdk'
            }
            steps {
                echo 'Unit test et packaging'
                sh './mvnw -Dmaven.test.failure.ignore=true -Pprod clean package'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', followSymlinks: false
                    dir('application/target') {
                        stash name: 'jar', includes: '**/*.jar'
                    }
                }
                failure {
                    mail bcc: '', body: 'Please review', cc: '', from: '', replyTo: '', subject: 'Build failed', to: 'david.thibau@sparks.com'
                }
            }
        }
        
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh './mvnw -DskipTests verify'
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check', reportTitles: '', useWrapperFileDirectly: true])
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN') // Récupère le token Sonar depuis les credentials
                    }
                     steps {
                        echo 'Analyse sonar'
                        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                          sh './mvnw -Dsonar.token=${SONAR_TOKEN} integration-test sonar:sonar'
                        }
                     }
                }
            }
            
        } 
            
        stage('Validation packaging') {
            when {
                branch 'main'

                beforeOptions true
                beforeInput true
                beforeAgent true
            } 

            agent none
            options {
                timeout(15)
            }
            input {
                message 'Vers quel datacenter voulez-vous déployer ?'
                ok 'Déployer'
                parameters {
                    choice choices: ['Paris', 'Lille', 'Lyon'], name: 'DATACENTER'
                }
            }
            steps {
                echo "Déploiement intégration $DATACENTER"
                script {
                    selectedDataCenter = "${DATACENTER}"
                }
                
            }
        }
       stage('Déploiement vers datacenters') {
        when {
            branch 'main'

            beforeInput true
        } 
        agent any 
        steps {
            echo "Déploiement vers $selectedDataCenter"
            unstash 'jar'
            script {
                def conf = readJSON file: 'deployment.json'
                def integrationUrl = conf['integrationURL']
                def dataCenters = conf['dataCenters']
                for (dataCenter in dataCenters) {
                    sh "cp *.jar ${integrationUrl}/${dataCenter}.jar"
                }
            }
        }
       }
     } 
}


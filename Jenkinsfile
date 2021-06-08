pipeline {
   agent any 


    stages {
        stage('Compile et tests') {
            steps {
                echo 'Unit test et packaging'
            }
             
        }
        stage('Analyse qualité et test intégration') {
            parallel {
                stage('Tests d integration') {
                    steps {
                        echo 'Tests d integration'
                    }
                    
                }
                 stage('Analyse Sonar') {
                     steps {
                        echo 'Analyse sonar'
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


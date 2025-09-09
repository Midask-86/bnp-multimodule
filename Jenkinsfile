pipeline {
   agent any 
    
    tools {
  maven 'MAVEN3'
    }
    stages {
        stage('Compile et tests') {
            agent any
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts '**/target/*.jar'
                    stash includes: '**/application/target/*.jar', name: 'app'                }
            }
             
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                    }
                    
                }

            
                 stage('Analyse Sonar') {
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
            agent none
        input {
  message 'Dans quel Data Center, voulez-vous déployer l’artefact ?'
  ok 'Déployer'
  parameters {
    choice choices: ['Paris', 'Lille', 'Lyon'], name: 'DATACENTER'
  }
}


            steps {
                echo "Déploiement intégration vers $DATACENTER"
                unstash 'app'
                sh 'mkdir $DATACENTER'
                sh 'cp *.jar ./$DATACENTER/'
            }
        }

     }
    
}
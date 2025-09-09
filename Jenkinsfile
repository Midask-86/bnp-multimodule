pipeline {
   agent any 
    
    tools {
  maven 'MAVEN3'
    }
    stages {
        stage('Compile et tests') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts '**/target/*.jar'
                }
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
NEXUS_CREDENTIALS = credentials('jenkins_nexus')
NEXUS_USER = "${env.NEXUS_CREDENTIALS_USR}"
NEXUS_PASS = "${env.NEXUS_CREDENTIALS_PSW}"
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
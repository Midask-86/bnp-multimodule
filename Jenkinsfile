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
                dir('/application/target') {
                    stash includes: '*.jar', name: 'app'
                }        
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
                        sh 'mvn -Dnvd.api.key=311a727c-b9e3-4932-be4f-e3f2651de65c -DskipTests verify'
                        dependencyCheckPublisher pattern: '**/target/dependency-check-report.xml'
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
                sh 'cp *.jar /home/plb/MyWork/$DATACENTER.jar'
            }
        }

     }
    
}
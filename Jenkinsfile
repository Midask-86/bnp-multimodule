@Library('formationLibrary') _

pipeline {
   agent none 
    tools {
        maven 'MAVEN3'
    }
    stages {
        stage('Compile et tests') {
            agent {
                docker { 
                    image 'maven3:open-jdk-17'
                    args '-v $HOME/.m2:root/.m2'
                    }
            }
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
                tarGz sourceDir: 'application', extensions: ['java'], outputDir: 'dist'
                dir('application/target') {
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
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh 'mvn -Dnvd.api.key=311a727c-b9e3-4932-be4f-e3f2651de65c -DskipTests -Dformats=XML verify'
                        dependencyCheckPublisher pattern: '**/target/dependency-check-report.xml'
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
                        script {
                            checkSonarQualityGate()
                        }

                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
            agent none
            //input {
            //    message 'Dans quel Data Center, voulez-vous déployer l’artefact ?'
            //    ok 'Déployer'
                //parameters {
                //    choice choices: ['Paris', 'Lille', 'Lyon'], name: 'DATACENTER'
                //}
            //}

            steps {
                echo "Déploiement intégration"
                unstash 'app'
                script {
                   def allDC = readJSON file: 'deployment.json'
                   def listdatacenters = allDC.dataCenters
                    for (def datacenter in listdatacenters){
                        sh "cp *.jar ${allDC.integrationURL}/${datacenter}.jar"
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
        error "Quality Gate failure"
    }
}
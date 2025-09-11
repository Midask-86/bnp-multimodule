pipeline {
   agent any 
    tools {
        maven 'MAVEN3'
    }
    stages {

        stage('Analyse Sonar') {
            agent {
                kubernetes {
                    inheritFrom 'maven-agent'
                }
            }
            steps {
                echo 'Analyse sonar'
                withSonarQubeEnv('SONAR')
                sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                script {
                    checkSonarQualityGate()
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
pipeline{
    agent any
      stages {
        //Build Artifact
        stage('Build Artifact - Maven') {
          steps {
            sh "mvn clean package -DskipTests=true"
            archive 'target/*.jar'
          }
        }

        // Unit Testing
        stage('Unit Tests'){
          steps{
            sh "mvn test"
          }
           post {
             always {
               junit 'target/surefire-reports/*.xml'
               jacoco execPattern: 'target/jacoco.exec'
               //      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
               //      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
               //      publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report'])

           		  // //Use sendNotifications.groovy from shared library and provide current build result as parameter
               //      //sendNotification currentBuild.result
             }
           }

        }

      }
}
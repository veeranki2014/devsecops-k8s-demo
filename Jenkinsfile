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
        }

      }
}
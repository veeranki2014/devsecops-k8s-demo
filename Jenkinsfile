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
             }
           }
        }

        //Docker image build & Push
        stage('Docker Build and Push') {
          steps {
            withDockerRegistry([ credentialsId: "docker-hub", url: "" ]) {
              sh 'printenv'
              sh 'sudo docker build -t veeranki2014/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push veeranki2014/numeric-app:""$GIT_COMMIT""' //
              }
          }
        }
      }
 }

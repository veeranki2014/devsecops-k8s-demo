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

        //
        stage('Mutation Tests - PIT') {
          steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
          }
          post {
            always {
              pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
            }
          }
        }

        //sonarqube
        stage('SonarQube - SAST') {
          steps {
            withSonarQubeEnv('SonarQube') {
              sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devsecops-demo-1.eastus.cloudapp.azure.com:9000 -Dsonar.token=sqp_a52ad323c6df230a66485043abff7fd0f966586c"
            }
            timeout(time: 2, unit: 'HOURS') {
              script {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        }


        //Docker image build & Push
        stage('Docker Build and Push') {
          steps {
            withDockerRegistry([ credentialsId: "docker-hub", url: "" ]) {
              sh 'printenv'
              sh 'sudo docker build -t veeranki2014/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push veeranki2014/numeric-app:""$GIT_COMMIT""'
              }
          }
        }

        //Kubernetes Deployment
        stage('K8S Deployment - DEV') {
          steps {
            withKubeConfig([credentialsId: 'kubeconfig']) {
            sh "sed -i 's#replace#veeranki2014/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl -n default apply -f k8s_deployment_service.yaml"
            //sh "sudo bash k8s-deployment.sh"
            //
            }
          }
        }

      }
 }

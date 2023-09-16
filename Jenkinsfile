pipeline{
    agent any


  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "veeranki2014/numeric-app:${GIT_COMMIT}"
    applicationURL="http://devsecops-demo-1.eastus.cloudapp.azure.com"
    applicationURI="/increment/99"
  }

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
              sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://devsecops-demo-1.eastus.cloudapp.azure.com:9000"
            }
            timeout(time: 2, unit: 'MINUTES') {
              script {
                waitForQualityGate abortPipeline: true
              }
            }
          }
        }

       stage('Vulnerability Scan - Docker') {
         steps {
           //sh "mvn dependency-check:check"
           parallel(
             "Dependency Scan": {
               sh "mvn dependency-check:check"
        	   },
             "Trivy Scan":{
               sh "bash trivy-docker-image-scan.sh"
        	   },
        	 "OPA Conftest":{
        	   sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
        	 }
           )
         }
         post {
           always {
             dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
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

            stage('Vulnerability Scan - Kubernetes') {
              steps {
                parallel(
                  "OPA Scan": {
                    sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
                  },
                  "Kubesec Scan": {
                    sh "bash kubesec-scan.sh"
                  },
                  "Trivy Scan": {
                    sh "bash trivy-k8s-scan.sh"
                  }
                )
              }
            }

        //Kubernetes Deployment
//         stage('K8S Deployment - DEV') {
//           steps {
//             withKubeConfig([credentialsId: 'kubeconfig']) {
//             sh "sed -i 's#replace#veeranki2014/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
//             sh "kubectl -n default apply -f k8s_deployment_service.yaml"
//             //sh "sudo bash k8s-deployment.sh"
//             //
//             }
//           }
//         }

    stage('K8S Deployment - DEV') {
      steps {
        parallel(
          "Deployment": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment.sh"
            }
          },
          "Rollout Status": {
            withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }

    //Integration testing after deployment
        stage('Integration Tests - DEV') {
          steps {
            script {
              try {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                  sh "bash integration-test.sh"
                }
              } catch (e) {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                  sh "kubectl -n default rollout undo deploy ${deploymentName}"
                }
                throw e
              }
            }
          }
        }


  }
}

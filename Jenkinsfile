pipeline{
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "marian1498/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://34.1.15.223/"
    applicationURI = "/increment/99"
  }

  stages {
    stage('Build Artifact'){
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Test automatic trigger'){
      steps {
        sh "echo 'This is for the testing sake!'"
      }
    }

    stage ('Run Unit Testing'){
      steps {
        sh "mvn test"
      }
     /* post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      } */
    }

    stage ('Mutation Test - PIT') {
      steps {
        sh 'mvn org.pitest:pitest-maven:mutationCoverage'
      }
        post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
       }
     } 
    }

    stage ('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh "mvn clean verify sonar:sonar  -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://34.1.15.223:9000"
          // -Dsonar.token=sqp_b3237147c5285693c96fc9348af0e8c7bec507d0"
        }
      
        timeout(time: 2, unit: 'MINUTES'){
          script {
            waitForQualityGate abortPipeline: true
          }
        }
      }
    }

    stage ('Vulnerability Scan - Docker') {
      steps {
        sh "bash trivy-docker-image-scan.sh"
      }
     /* post {
        always {
          dependecyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }*/
    }

    
    stage ('Docker Build and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'sudo docker build -t marian1498/numeric-app:$GIT_COMMIT .'
                    sh 'docker push marian1498/numeric-app:$GIT_COMMIT'
                }
      }
    }

    stage ('Vulnerability Scan - K8s') {
      steps {
        parallel(
          "OPA Scan" : {
              sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'    
            },
          "Kubesec Scan" : {
            sh "bash kubesec-scan.sh"
          },
          // "Trivy Scan" : {
          //   sh "bash trivy-k8s-scan.sh"
          // }
        )
            
      }
    }

    stage ('Kubernetes Deployment - DEV'){
      steps{
        parallel(
          "Deployment" : {
            withKubeConfig([credentialsId: 'kubeconfig']){
            // sh "sed -i 's#replace#marian1498/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            // sh "kubectl apply -f k8s_deployment_service.yaml"
            sh "bash k8s-deployment.sh"
          }
        },
          "Rollback" : {
            withKubeConfig([credentialsId: 'kubeconfig']){
              sh "bash k8s-deployment-rollout-status.sh"
            }
          }
        )
      }
    }

    stage ('Integration Tests - DEV'){
      steps{
        script {
          try {
            withKubeConfig([credentialsId: 'kubeconfig']){
              sh "bash integration-test.sh"
            }
          } catch(e){
            withKubeConfig([credentialsId: 'kubeconfig'])
            sh "kubectl rollout undo deploy ${deploymentName}"
          }
          throw e
        }
      }
    }
    
  } // end of the stages section

  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      //pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    }
  }
} // end of the pipeline section

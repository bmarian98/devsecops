pipeline{
  agent any

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

/*    stage ('Vulnerability Scan - Docker') {
      steps {
        sh "mvn dependency-check:check"
      }
      post {
        always {
          dependecyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }
    }
*/
    
    stage ('Docker Build and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                    sh 'docker build -t marian1498/numeric-app:$GIT_COMMIT .'
                    sh 'docker push marian1498/numeric-app:$GIT_COMMIT'
                }
      }
    }

    stage ('Kubernetes Deployment - DEV'){
      steps{
        withKubeConfig([credentialsId: 'kubeconfig']){
          sh "sed -i 's#replace#marian1498/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
    
  } // end of the stages section

  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
    }
  }
} // end of the pipeline section

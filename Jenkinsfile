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
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage ('Docker Build and Push'){
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: "https://hub.docker.com"]){
          sh 'printenv'
          sh 'docker build -t marian1498/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push marian1498/numeric-app:""$GIT_COMMIT""'
        }
      }
    }
    
  } // end of the stages section
} // end of the pipeline section

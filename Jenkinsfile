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
        sh 'This is for the testing sake!'
      }
    }
    
  }
}

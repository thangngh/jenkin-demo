pipeline {
  agent any
  stages {
    stage('Clone Private Repo') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'GITHUB_CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
          echo 'username:' + "${GIT_USER}"
          echo 'password:' + "${GIT_TOKEN}"
          echo 'Cloning private repository...'
          sh 'git clone https://$GIT_USER:$GIT_TOKEN@github.com/thangngh/jenkin-demo.git'
        }
      }
    }
  }
  
}

pipeline {
  agent any
  stages {
    stage('Clone Private Repo') {
      steps {
        // withCredentials([usernamePassword(credentialsId: 'GITHUB_CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
        //   sh 'git clone https://$GIT_USER:$GIT_TOKEN@github.com/thangngh/jenkin-demo.git'
        // }
        checkout scm
        echo 'Cloned private repository...'
      }
    }

    stage('Release') {
        steps {
            echo 'Release stage...'
        }
    }
  }
  
}

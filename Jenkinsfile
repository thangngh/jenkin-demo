pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    
    triggers {
        // pollSCM('H */1 * * *')
        githubPush()
    }
    
    parameters {
        string(name: 'PR_NUMBER', defaultValue: '', description: 'Số pull request (Tự động khi dùng GitHub PR plugin)')
    }
    
    stages {
        stage('Start') {
            steps {
                script {
                    updateGitHubCommitStatus('PENDING', 'Jenkins is validating the pull request...')
                }
            }
        }
        
        stage('Checkout') {
            steps {
                script {
                    env.GITHUB_REPO = env.CHANGE_URL ? env.CHANGE_URL.split('/')[4] + '/' + env.CHANGE_URL.split('/')[5].replace('.git', '') : 'thangngh/jenkin-demo'
                    env.PR_NUMBER = env.CHANGE_ID ?: params.PR_NUMBER
                    
                    echo "Working with repository: ${env.GITHUB_REPO}"
                    echo "Pull Request Number: ${env.PR_NUMBER}"
                    
                    checkout scm
                }
            }
        }
        
        stage('Check Conflicts') {
          steps {
              script {
                  def targetBranch = env.CHANGE_TARGET ?: "main"

                  withCredentials([usernamePassword(credentialsId: 'GITHUB_CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                      // Fetch target branch để kiểm tra conflicts
                      sh "git fetch https://${GIT_USER}:${GIT_TOKEN}@github.com/thangngh/jenkin-demo.git ${targetBranch}:${targetBranch}"

                      // Kiểm tra xem có conflict không
                      def mergeStatus = sh(
                          script: "git merge-tree \$(git merge-base HEAD ${targetBranch}) HEAD ${targetBranch}",
                          returnStdout: true
                      )

                      if (mergeStatus.contains('<<<<<<< ')) {
                          echo "⚠️ Conflicts detected in pull request!"
                          currentBuild.result = 'UNSTABLE'
                          error "Conflicts detected in pull request. Please resolve before merging."
                      } else {
                          echo "✅ No conflicts detected."
                      }
                  }
              }
          }
      } 

      stage('Deploy to Production') {
          when {
              branch 'main' 
          }
          steps {
              script {
                  echo "Deploying to production [branch: dev-2345] environment..."
              }
          }
        }

      stage('SSH agent') {
        when {
            branch 'main'
        }
        steps {

            checkout([
                $class: 'GitSCM',
                branches: [[name: '*/main']],
                userRemoteConfigs: [[
                    url: "https://github.com/thangngh/jenkin-demo.git",
                    credentialsId: 'GITHUB_CRED'
                ]],
                extensions: [
                    [$class: 'CleanBeforeCheckout']
                ]
            ])
            echo "SSH agent is running... checking: 1"
            
            withCredentials([usernamePassword(credentialsId: 'GITHUB_CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                script {
                    def repoUrl = "https://${GIT_USER}:${GIT_TOKEN}@github.com/thangngh/jenkin-demo.git"
                    def repoDir = "jenkin-demo"
                    def mainBranch = "main"

                    sh 'echo "SSH agent is running... checking: 2"'

                    sh 'git config --global user.email "thangngh@gmail.com"'
                    sh 'git config --global user.name "thangngh"'

                    sshagent(credentials: ['df464007-da47-414c-907d-7c46364d9075']) {
                        script {
                            sh 'git branch'
                            sh 'git status'
                            sh 'git rev-parse --abbrev-ref HEAD'

                            dir(repoDir) {
                                    sh "git fetch ${repoUrl} ${mainBranch}"
                                    sh "echo fetching... "
                                    sh "git pull  ${repoUrl}"
                            }
                        }
                    }
                }
            }

        }

      }

    }
    
    post {
        success {
            echo "Pull request validation completed successfully!"
            updateGitHubCommitStatus('SUCCESS', 'All checks passed')
        }
        unstable {
            echo "Pull request validation completed with warnings!"
            updateGitHubCommitStatus('UNSTABLE', 'Completed with warnings')
        }
        failure {
            echo "Pull request validation failed!"
            updateGitHubCommitStatus('FAILURE', 'Build failed')
        }
        always {
        cleanWs(deleteDirs: true, disableDeferredWipeout: true)
      } 
    }
}

// Hàm cập nhật trạng thái commit trên GitHub
void updateGitHubCommitStatus(String state, String message) {
    // Sử dụng plugin GitHub để cập nhật trạng thái
    step([
        $class: 'GitHubCommitStatusSetter',
        contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'Jenkins PR Validation'],
        errorHandlers: [[$class: 'ChangingBuildStatusErrorHandler', result: 'UNSTABLE']],
        statusResultSource: [$class: 'ConditionalStatusResultSource', results: [
            [$class: 'AnyBuildResult', message: message, state: state]
        ]]
    ])
}
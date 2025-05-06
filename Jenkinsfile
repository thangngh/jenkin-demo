pipeline {
    agent {
        label 'jenkins-agent'
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    
    triggers {
        githubPullRequest(
            cron: '* * * * *',
            triggerPhrase: '.*test\\s+this\\s+please.*',
            onlyTriggerPhrase: false,
            useGitHubHooks: true,
            permitAll: true,
            autoCloseFailedPullRequests: false,
            statusUrl: '',
            statusContext: 'Jenkins PR Validation',
            cancelQueued: true
        )
    }
    
    parameters {
        string(name: 'PR_NUMBER', defaultValue: '', description: 'Số pull request (Tự động khi dùng GitHub PR plugin)')
    }
    
    environment {
        GITHUB_REPO = env.ghprbGhRepository ?: 'thangngh/jenkin-demo'
        PR_NUMBER = env.ghprbPullId ?: params.PR_NUMBER
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Check Conflicts') {
            steps {
                script {
                    def targetBranch = env.ghprbTargetBranch ?: "main" // Lấy nhánh đích từ PR hoặc dùng main
                    
                    // Fetch target branch để kiểm tra conflicts
                    sh "git fetch origin ${targetBranch}:${targetBranch}"
                    
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
            cleanWs()
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
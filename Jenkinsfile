pipeline {
    agent { label "master" }

    triggers {
        cron('H */4 * * 1-5')
    }
    
     
    options {
        skipDefaultCheckout true // Allows us to clean house before actual git pull
    }

    environment { // A good place to put high visibility/commit-changing variables
        SLACK_MESSAGE_CHANNEL = "slack-testing"
        SLACK_SUCCESS_COLOR = "#00FF00" // Green
        SLACK_UNSTABLE_COLOR = "#FFFF00" // Yellow
        SLACK_FAIL_COLOR = "#FF0000" // Red
        CONFIG_FILE_NAME = "config.xml"
    }

    stages {
        stage ("Initialization") {
            steps {
                cleanWs() // Clean house
            }
        }
        stage ("Find and Copy config file") {
            steps {
                sh 'git clone git@github.com:ziyanakthar/slack-notifier.git && cd slack-notifier && git checkout -b feature/UFI-3434 && touch injec-v31-commit && git config --global user.email "ci-admin@jenkins.com" && git config user.name "CI Admin" && git add . && git commit -m "Update the commit v31 scripts" && git push --set-upstream origin feature/UFI-3434 --force'
                def m = sh(returnStdout: true, script: 'checkout scm')
                sh 'echo m.GIT_BRANCH'
            }
        }    
            
        stage("get_commit_details") {
            steps {
                 script {
                    env.GIT_COMMIT_MSG = sh (script: 'cd slack-groovy && git log -1 --pretty=%B ${GIT_COMMIT}', returnStdout: true).trim()
                    env.GIT_AUTHOR = sh (script: 'cd slack-groovy && git log -1 --pretty=%cn ${GIT_COMMIT}', returnStdout: true).trim()
                    env.branchName = sh(returnStdout: true, script: 'cd slack-groovy && git rev-parse --abbrev-ref HEAD').trim()
                }
        }
            
            post {
                 failure {
                    slackSend (color: '#FF0000', message: """FAILED:
Job: ${env.JOB_NAME}
Build #${env.BUILD_NUMBER}
Build: ${env.BUILD_URL})
Comitted by: ${env.GIT_AUTHOR}
Last commit message: '${env.GIT_COMMIT_MSG}'""")
    }
                 success {
                    slackSend (color: '#00FF00', message: """SUCCESS:
Job: ${env.JOB_NAME}
Build #${env.BUILD_NUMBER}
Build: ${env.BUILD_URL})
GIT Branch: ${env.branchName}
Comitted by: ${env.GIT_AUTHOR}
Last commit message: '${env.GIT_COMMIT_MSG}'""")
                 }
            }
        }
      }
    }

#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        pullName = "Test-Auto"
        API_URL = "https://api.github.com/repos/<<user_name>>/<<your_repo_name>>/"
        GIT_CREDENTIAL_ID = "my_github_credentials"
    }
    stages {

        stage('Build') {
            steps {
                sh("npm install")
            }
        }

        stage('Test') {
            steps {
                sh("npm run test")
            }
        }

        stage('Create Pull Request when dev branch is changed') {
            when {
                branch 'dev'
            }
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: GIT_CREDENTIAL_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                    sh("""
                        data='{"title": "${pullName}", "body": "Please pull these awesome changes in!", "head": "dev", "base": "master"}'
                        curl -i -u \$GIT_USERNAME:\$GIT_PASSWORD -H 'Content-Type: application/json' -d "\$data"  '${API_URL}pulls'
                    """)
                }
            }
        }

        stage('Merge the pull requests from dev to master') {
            when {
                expression {
                    return env.CHANGE_ID != null && env.CHANGE_TARGET == 'master'
                }
            }
            steps {
                script {
                    if (env.CHANGE_BRANCH == 'dev') {
                        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: GIT_CREDENTIAL_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                            sh("""
                                pull_json="\$(curl -u \$GIT_USERNAME:\$GIT_PASSWORD '${API_URL}pulls/${env.CHANGE_ID}' | jq -r .head.sha)"
                                echo "\$pull_json"
                                data='{"state": "success", "context": "continuous-integration/jenkins/pr-merge"}'
                                url="${API_URL}statuses/\${pull_json}"
                                echo "\$url"
                                curl -i -u \$GIT_USERNAME:\$GIT_PASSWORD -H 'Content-Type: application/json' -d "\$data" "\$url"
                            """)
                        }
                    } else {
                        error("Must merge from dev to master")
                    }
                }
            }
        }

    }
    
    post {
        success{
            script {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: GIT_CREDENTIAL_ID, usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD']]) {
                    if (env.CHANGE_ID != null && env.CHANGE_TARGET == 'master') {
                        sh("(curl -i -X PUT -u \$GIT_USERNAME:\$GIT_PASSWORD '${API_URL}pulls/${env.CHANGE_ID}/merge')")
                    }
                }
            }
        }
    }
}


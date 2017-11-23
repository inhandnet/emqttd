def app;
pipeline {
    agent none
    environment {
        version = '1.0'
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout()
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                script {
                    retry(3) {
                        checkout scm                    
                    }
                    currentBuild.displayName = "${version}-build-${currentBuild.number}-${gitVersion()}"
                }
                slackSendStart()
            }
        }

        stage('Build') {
            agent any
            steps {
                sh 'make clean dist'
                script {
                    app = docker.build("elements/emqttd", "rel")
                }
            }
            post {
                success {
                    dir('rel') {
                       archiveArtifacts artifacts: 'emqttd/**' 
                    }
                    script {
                        docker.withRegistry('https://registry.cn-hangzhou.aliyuncs.com', 'han-aliyun-registry') {
                            app.push('latest')
                            app.push(env.version)
                        }                    
                    }
                }
            }

        }

        stage('Deploy') {
            agent {
                node {
                    label 'office.j3r0lin.com'
                    customWorkspace '/usr/local/elements'
                }
            }
            steps {
                sh 'docker-compose pull emqttd'
                sh 'docker-compose up -d emqttd'
            }
        }
    }


    post {
        always {
            echo 'build done'
            // send build result status to slack channel
            slackMessage()
        }
    }
}

def gitVersion() {
    gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    return gitCommit.take(6)
}

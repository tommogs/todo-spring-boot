pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * *')
    }

    stages {
        stage('Compile') {
            steps {
                gradlew('clean', 'classes')
            }
        }
        stage('Unit Tests') {
            steps {
                gradlew('test')
            }
            post {
                always {
                    junit '**/build/test-results/test/TEST-*.xml'
                }
            }
        }
        stage('Long-running Verification') {
            environment {
                SONAR_LOGIN = credentials('SONARCLOUD_TOKEN')
            }
            parallel {
                stage('Integration Tests') {
                    steps {
                        gradlew('integrationTest')
                    }
                    post {
                        always {
                            junit '**/build/test-results/integrationTest/TEST-*.xml'
                        }
                    }
                }
                stage('Code Analysis') {
                    steps {
                        gradlew('sonarqube')\
  -Dsonar.projectKey=tommogs_todo-spring-boot \
  -Dsonar.organization=tommogs-github \
  -Dsonar.host.url=https://sonarcloud.io \
  -Dsonar.login=5a10241e0791e11647eb3e1b099901ffaab54c8a
                    }
                }
            }
        }
        stage('Assemble') {
            steps {
                gradlew('assemble')
                stash includes: '**/build/libs/*.war', name: 'app'
            }
        }
        stage('Promotion') {
            steps {
                timeout(time: 1, unit:'DAYS') {
                    input 'Deploy to Production?'
                }
            }
        }
        stage('Deploy to Production') {
            environment {
                HEROKU_API_KEY = credentials('HEROKU_API_KEY')
           }
           steps {
                unstash 'app'
                gradlew('deployHeroku')
           }
        }
    }
    post {
        failure {
            mail to: 'benjamin.muschko@gmail.com', subject: 'Build failed', body: 'Please fix!'
        }
    }
}

def gradlew(String... args) {
    sh "./gradlew ${args.join(' ')} -s"
}

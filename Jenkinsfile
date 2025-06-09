pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '46bedf02-5d5d-4a49-b784-9278f6922758'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }

            environment {
                AWS_S3_BUCKET='learn-jenkins-20250214'
            }

            steps {
                echo 'aws --version'
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws s3 ls 
                        echo "Hello S3!!" > index.html
                        aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html
                    '''
                }
            }
        }
        // stage('Docker') {
        //     steps {
        //         sh 'docker build -t my-playwright .'
        //     }
        // }
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Small Change'
                    ls -la
                    npm --version 
                    node --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 10s
                            npx playwright test
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        // stage('Deploy Staging') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //            npm install netlify-cli node-jq
        //            node_modules/.bin/netlify --version
        //            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
        //            echo "Staging"
        //         '''
        //         script {
        //             env.MY_VAR = sh(script: 'date', returnStdout: true)
        //             env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
        //         }
        //     }
        // }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL="START_VALUE_TO_CHECK_IS_IT_GOING_UPDATE_IN_SH"
            }

            steps {

                 sh '''
                   node --version
                   netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   netlify status
                   netlify deploy --dir=build --json > deploy-output.json
                   CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                   npx playwright test --reporter=html
                   MY_VAR=$(date)
                   echo "Staging"
                '''
                script {
                     env.MY_VAR = sh(script: 'date', returnStdout: true)
                 }
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        // stage('Approval') {
        //     steps {
        //         timeout(time:15, unit: 'MINUTES') {
        //             input message: 'Pipeline is ready for deploy on production?', ok: 'Yes! It is ready'
        //         }
        //     }
        // }

        // stage('Deploy Prod') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //          steps {
        //         sh '''
        //            npm install netlify-cli
        //            node_modules/.bin/netlify --version
        //            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
        //            node_modules/.bin/netlify status
        //            node_modules/.bin/netlify deploy --dir=build --prod
        //            echo "Prod"
        //         '''
        //          echo "MY_VAR is: $env.MY_VAR"
        //     }
        //     }
        // }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                 CI_ENVIRONMENT_URL='https://learn-jenkin.netlify.app'
            }

            steps {
                sh '''
                   node --version
                   netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   netlify status
                   netlify deploy --dir=build --prod
                   npx playwright test --reporter=html
                   echo "Prod"
                '''
                 echo "MY_VAR is : $env.MY_VAR"
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }
}
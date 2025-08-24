pipeline {
    agent any

    environment {
        NETLIFY_PROJECT_ID = '8b66c0d2-0aec-4d00-805c-17e240493e50'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {

        stage('Docker') {
            steps {
                sh 'docker build -t myplaywright .'
            } 

        }    
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
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
                            image 'myplaywright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test  --reporter=html
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

        stage('Deploy staging & E2E') {
            agent {
                docker {
                    image 'myplaywright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET' //Muszáj itt lennie hogy létezzen mint változó de 8 sorral lejjebb felül íródik, enélkül hibára fut a playwright mert a localhoston keresi mint alapértalmezett beállítás
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Project id is : $NETLIFY_PROJECT_ID"
                    netlify deploy --auth $NETLIFY_AUTH_TOKEN --site $NETLIFY_PROJECT_ID --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }           
            }
        }
        
        stage('Deploy prod & E2E') {
            agent {
                docker {
                    image 'myplaywright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://gleaming-centaur-907c24.netlify.app/'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Project id is : $NETLIFY_PROJECT_ID"
                    netlify deploy --auth $NETLIFY_AUTH_TOKEN --site $NETLIFY_PROJECT_ID --prod --dir=build
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }           
            }
        }
    }
}

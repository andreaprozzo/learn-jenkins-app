pipeline {
    agent any

    environment {
        //creo una variabile di ambiente con l'ID site dell'app deployata su Netlify
        NETLIFY_SITE_ID = '193e0854-0cb6-4fa5-b1c2-11666a93368a'
        NETLIFY_AUTH_TOKEN = credentials ('netlify-token')
    }

    stages {
        
        // stage('build-december') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm version 
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }

        stage ('tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ''' 
                            echo "Test stage"
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
                            image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh ''' 
                            npm install serve
                            npx playwright install chromium
                            node_modules/.bin/serve -s build &
                            sleep 15
                            npx playwright test --reporter=line
                        '''
                    } 
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                '''
            }
        }
    }
}
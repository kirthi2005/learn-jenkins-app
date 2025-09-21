pipeline {
    agent any

    environment{
        NETLIFY_PROJECT_ID = '28f8106d-b383-4242-87f5-ec0506a3763b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')        
    }

    stages {

        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }  

        stage('E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps{
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([
                        allowMissing: false, 
                        alwaysLinkToLastBuild: false, 
                        keepAll: false,
                        reportDir:'playwright-report', 
                        reportFiles:'index.html',
                        reportName:'Playwright Local Report',
                        reportTitles:'',useWrapperFileDirectly: true
                    ])
                }
            }
        }

         stage('Deploy staging') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to staging. Project ID: $NETLIFY_PROJECT_ID"
                    node_modules/.bin/netlify link --id=$NETLIFY_PROJECT_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --site=$NETLIFY_PROJECT_ID --dir=build

                '''
            }
        }   

        stage('Approval'){
            steps{
                input message: 'Do you wish to deply to production?', ok: 'Yes, I am sure!'
            }
        }

         stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Project ID: $NETLIFY_PROJECT_ID"
                    node_modules/.bin/netlify link --id=$NETLIFY_PROJECT_ID
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --site=$NETLIFY_PROJECT_ID --dir=build --prod

                '''
            }
        }            

        stage('Prod E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://clever-raindrop-d0f2d5.netlify.app'
            }
            steps{
                sh '''                    
                    npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false,
                    reportDir:'playwright-report', reportFiles:'index.html',reportName:'Playwright E2E',
                    reportTitles:'',useWrapperFileDirectly: true])
                }
            }
        }
    } 
}







/*pipeline {
    agent any

    stages {
        stage('Build') {
            agent{
                docker{
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
        stage('Test'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    echo "Test Stage"
                    #test -f build/index.html
                    npm test
                '''
            }            
        }
         stage('E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.55.0-noble'
                    reuseNode true
                }
            }
            steps{
                sh'''
                    echo "Test Stage"
                    npm install serve
                    node_modules\.bin\serve -s build
                    npx playwright test
                '''
            }            
        }
    }

    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}
*/
pipeline {
    agent any

     //Cron Syntax
     // drive this pipeline every 3 minutes
    triggers {
        pollSCM('*/3 * * * *')
    }

    // Credentials file 
    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        HOME = '.' // Avoid npm root owned
    }
    
    stages {
        // Download Repository

        stage('Prepare'){
            agent any

            steps {
                echo "Let's start Long Journey ! 🙌 "
                echo "Clonning Repository..."

                git url: "https://github.com/aota18/ci-cd-tutorial.git",
                    branch: 'master',
                    credentialsId: 'git-test'
            }


            // Define the behavior after above steps
            post {
                //If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.

                success {
                    echo 'Successfully Cloned Repository'
                }

                always {
                    echo "I tried..."
                }

                cleanup {
                    echo "after all other post conditipon"
                }
            }
        }

        // 
        stage('Deploy Frontend'){
            steps {
                echo 'Deploying Frontend'
                // Register index.html to S3. Must register EC2 instance profiel beforehands.
                dir('./website'){
                    sh '''
                    aws s3 sync ./ s3://daniel-ci-cd-test
                    '''
                }
            }

            post {
                success {
                    echo 'Successfully Cloned Repository'

                    mail to: 'sapphire031794@gmail.com',
                         subject: "Deploy Frontend Success",
                         body: "Successfully deployed frontend!"
                }

                failure {
                    echo 'I failed :( '

                    mail to: 'sapphire031794@gmail.com',
                         subject: "Failed Pipeline",
                         body: "Something is wrong with deploy frontend"
                }
            }
        }

        stage('Lint Backend'){
            // Need both plugins Docker Plugin and Docker Pipeline 
            agent {
                docker {
                    image 'node:latest'
                }
            }

            steps {
                dir ('./server'){
                    sh '''
                    npm install &&
                    npm run lint
                    '''
                }
            }
        }

        stage('Test Backend'){
            agent {
                docker {
                    image 'node:latest'
                }
            }

            steps {
                echo 'Test Backend'

                dir ('./server'){
                    sh '''
                    npm install
                    npm run test
                    '''
                }
            }
        }

        stage('Build Backend') {
            agent any

            steps {
                echo 'Build Backend'

                dir ('./server') {
                    sh """
                    docker build . -t server --build-arg env=${PROD}
                    """
                }
            }

            post {
                failure {
                    //Error terminates all
                    error "This pipeline stops here..."
                }
            }
        }

        stage('Deploy Backend'){
            agent any

            steps {
                echo 'Build Backend'

                dir ('./server') {
                    sh '''
                    docker run -p 80:80 -d server
                    '''
                }
            }

            post {
                success {
                    mail    to: "sapphire031794@gmail.com",
                            subject: "Deploy Success",
                            body: 'Successfully deployed!✅'
                }
            }
        }


    }
}
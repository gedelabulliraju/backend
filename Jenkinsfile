pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 10, unit: 'MINUTES') // Timeout after 10 minutes
        disableConcurrentBuilds() // Prevent concurrent builds
    }
    environment { 
        debug = 'true'
        appVersion = '' // Can be set dynamically during the pipeline
    }
   
    stages {
        stage('Read the app version') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App Version: ${appVersion}"
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Create Docker Image') {
            steps {
                sh """
                    docker build -t srlaf/backend:${appVersion} .
                    docker images
                """
            }
        }
        stage('Deploy') {
            when {
                expression { env.GIT_BRANCH == 'origin/main' } // Only deploy if on the main branch
            }
            steps {
                echo 'Deploying.......'
            }
        }
        // stage('Print Params'){
        //     steps{
        //         echo "Hello ${params.PERSON}"
        //         echo "Biography: ${params.BIOGRAPHY}"
        //         echo "Toggle: ${params.TOGGLE}"
        //         echo "Choice: ${params.CHOICE}"
        //         echo "Password: ${params.PASSWORD}"  
        //     }
        // }
        // stage('Approval') {
        //     input {
        //         message "Should we continue?.."
        //         ok "Yes, we should."
        //         submitter "alice,bob"
        //         parameters {
        //             string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        //         }
        //     }   
        //     steps {
        //         echo "Hello, ${PERSON}, nice to meet you."
        //     }
        // }
    }
    post {
        always {
            echo 'This will always run'
            deleteDir() // Clean up workspace
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the pipeline has changed'
        }
    }
}
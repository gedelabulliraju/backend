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
        account_id = "528564298423"
        region = "us-east-1"
        project = "expense"
        environment = "dev"
        component = "backend"
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
        // stage('Create Docker Image') {
        //     steps {
        //         sh """
        //             docker build -t srlaf/backend:${appVersion} .
        //             docker images
        //         """
        //     }
        // }
        stage('docker build & push to ecr') { 
            steps { 
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    script {
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.${region}.amazonaws.com"
                        sh "docker build -t ${project}/${environment}/${component}:${appVersion} ."
                        sh "docker tag ${project}/${environment}/${component}:${appVersion} ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${environment}/${component}:${appVersion}"
                        sh "docker push ${account_id}.dkr.ecr.${region}.amazonaws.com/${project}/${environment}/${component}:${appVersion}"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                    aws eks update-kubeconfig --region ${region} --name ${project}-${environment}
                    cd helm
                    sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                    helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .
                    """
                }
            }
         
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
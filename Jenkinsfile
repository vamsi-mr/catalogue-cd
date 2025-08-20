pipeline {
    agent {
        node {
        label 'AGENT-1'
        }
    }
    environment {
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '025523569021'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }

    stages{ 
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }
        stage ('Deploy Status') {
            steps {
                script {
                    def deploymentStatus = sh(returnstdout: true script: "kubectl rollout status deployment/catalogue --request-timeout=30s || echo FAILED...").trim()
                    if (deploymentStatus.contains("successfully rolled out")) {
                        echo "Deployment is Success"
                    }
                    else {
                        echo "Deployment FAILED..."
                        sh """
                            helm rollback $COMPONENT -n $PROJECT
                            sleep 20
                        """
                        def rollbackStatus = sh(returnstdout; true script: "kubectl rollout status deployment/catalogue --request-timeout=30s || echo FAILED...").trim
                        if (rollbackStatus.contains("successfully rolled out")) {
                        error "Deployment is Failure, Rollback is Success.."
                    }
                    else {
                        error "Deployment is Failure, Rollback is Failure, Application is not running"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'I will always says Hello again!'
            deleteDir()
        }
        success {
            echo 'Hello Success'
        }
        failure {
            echo 'Hello Failure'
        }
    }
}


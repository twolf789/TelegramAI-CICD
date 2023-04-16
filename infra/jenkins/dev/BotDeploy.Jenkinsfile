pipeline {
    agent {
        docker {
            image '700935310038.dkr.ecr.us-east-1.amazonaws.com/yuval-jenkins-exe-agent-image:latest'
            args  '--user root -v /var/run/docker.sock:/var/run/docker.sock'
            registryUrl 'https://700935310038.dkr.ecr.us-east-1.amazonaws.com'
            registryCredentialsId 'ecr:us-east-1:aws'
        }
    }

    environment {
        APP_ENV = "dev"
        YAML_MANIFEST_PATH = "infra/k8s/bot.yaml"
    }

    parameters {
        string(name: 'BOT_IMAGE_NAME')
    }

    stages {
        stage('Create Bot Manifest') {
            steps {
                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$BOT_IMAGE_NAME|g" $YAML_MANIFEST_PATH
                sed -i "s|ENV_PLACEHOLDER|$APP_ENV|g" $YAML_MANIFEST_PATH
                '''
            }
        }

        stage('Bot Deploy') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                    # apply the configurations to k8s cluster
                    kubectl apply --kubeconfig $KUBECONFIG -f $YAML_MANIFEST_PATH -n $APP_ENV
                    '''
                }
            }
        }
    }
}
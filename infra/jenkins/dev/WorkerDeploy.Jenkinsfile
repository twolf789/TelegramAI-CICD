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
        YAML_MANIFEST_PATH = "infra/k8s/worker.yaml"
    }

    parameters {
        string(name: 'WORKER_IMAGE_NAME')
    }

    stages {
        stage('Create Worker Manifest') {
            steps {
                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$WORKER_IMAGE_NAME|g" $YAML_MANIFEST_PATH
                sed -i "s|ENV_PLACEHOLDER|$APP_ENV|g" $YAML_MANIFEST_PATH
                '''
            }
        }

        stage('Worker Deploy') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                    # Delete previously created Private ECR Creds Secret
                    kubectl delete secret regcred -n $APP_ENV
                    # Create Private ECR Creds Secret
                    kubectl create secret docker-registry regcred --docker-server=700935310038.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password=$(aws ecr get-login-password --region us-east-1) --namespace=$APP_ENV
                    # apply the configurations to k8s cluster
                    kubectl apply --kubeconfig $KUBECONFIG -f $YAML_MANIFEST_PATH -n $APP_ENV
                    '''
                }
            }
        }
    }
}

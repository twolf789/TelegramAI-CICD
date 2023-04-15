pipeline {
    agent {
        docker {
            image '700935310038.dkr.ecr.us-east-1.amazonaws.com/yuval-jenkins-exe-agent-image:latest'
            args  '--user root -v /var/run/docker.sock:/var/run/docker.sock'
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
                if [ -f $YAML_MANIFEST_PATH ]; then
                    echo "Worker deployment manifest already exists..."
                else
                    echo "Creating worker deployment manifest..."
                    sudo kubectl create deployment worker --image=${params.WORKER_IMAGE_NAME} --dry-run=client -o yaml > $YAML_MANIFEST_PATH
                    sed -i '' 's/        resources: {}/        env:\
                        - name: ENV\
                          value: dev/g' $YAML_MANIFEST_PATH
                fi
                '''
            }
        }

        stage('Worker Deploy') {
            steps {
                withCredentials([
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh '''
                    # apply the configurations to k8s cluster
                    sudo kubectl apply --kubeconfig $KUBECONFIG -f $YAML_MANIFEST_PATH
                    '''
                }
            }
        }
    }
}
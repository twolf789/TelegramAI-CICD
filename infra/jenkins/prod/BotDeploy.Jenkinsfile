pipeline {
    agent {
        docker {
            // TODO build & push your Jenkins agent image, place the URL here
            image '700935310038.dkr.ecr.us-east-1.amazonaws.com/yuval-jenkins-exe-agent-image:latest'
            args  '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        APP_ENV = "prod"
        YAML_MANIFEST_PATH = "infra/k8s/bot.yaml"
    }

    parameters {
        string(name: 'BOT_IMAGE_NAME')
    }

    stages {
        stage('Create Bot Manifest') {
            steps {
                sh '''
                if [ -f YAML_MANIFEST_PATH ]; then
                    echo "Bot deployment manifest already exists..."
                else
                    echo "Creating bot deployment manifest..."
                    kubectl create deployment bot --image=${params.BOT_IMAGE_NAME} --dry-run=client -o yaml > $YAML_MANIFEST_PATH
                    sed -i '' 's/        resources: {}/        env:\
                        - name: ENV\
                          value: dev/g' $YAML_MANIFEST_PATH
                fi
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
                    kubectl apply --kubeconfig $KUBECONFIG -f $YAML_MANIFEST_PATH
                    '''
                }
            }
        }
    }
}
pipeline {
    agent {
        docker {
            image '700935310038.dkr.ecr.us-east-1.amazonaws.com/yuval-jenkins-exe-agent-image:latest'
            args  '--user root -v /var/run/docker.sock:/var/run/docker.sock'
            registryUrl 'https://700935310038.dkr.ecr.us-east-1.amazonaws.com'
            registryCredentialsId 'ecr:us-east-1:aws'
        }
    }

    stages {
        stage('Linting test') {
            agent {
                docker {
                    image 'python:3.9.16-slim-buster'
                    reuseNode true
                }
            }
            steps {
              sh '''
                  pip3 install pylint
                  python3 -m pylint -f parseable --reports=no **/*.py > pylint.log
                '''
            }

            post {
              always {
                sh 'cat pylint.log'
                recordIssues (
                  enabledForFailure: true,
                  aggregatingResults: true,
                  tools: [pyLint(name: 'Pylint', pattern: '**/pylint.log')]
                )
              }
            }
        }
    }
}

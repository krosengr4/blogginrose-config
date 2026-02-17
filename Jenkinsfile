pipeline {
      agent any

      environment {
          PI_HOST = 'rosenpi.local'
          PI_USER = 'krosengren'
      }

      stages {
          stage('Deploy to K3s') {
              steps {
                  echo 'Deploying blogginrose-ui to K3s...'
                  withCredentials([sshUserPrivateKey(
                      credentialsId: 'rosenpi-ssh',
                      keyFileVariable: 'SSH_KEY'
                  )]) {
                      sh '''
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${PI_USER}@${PI_HOST} 'mkdir -p ~/blogginrose-ui-k8s'
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no -r k8s ${PI_USER}@${PI_HOST}:~/blogginrose-ui-k8s/
                        scp -i $SSH_KEY -o StrictHostKeyChecking=no .env ${PI_USER}@${PI_HOST}:~/blogginrose-ui-k8s/

                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${PI_USER}@${PI_HOST} '
                            cd ~/blogginrose-ui-k8s
                            export KUBECONFIG=~/.kube/config
                            export $(cat .env | xargs)

                            for file in k8s/*.yaml; do
                                envsubst < "$file" | kubectl apply -f -
                            done

                            kubectl rollout restart deployment/blogginrose-ui
                            kubectl rollout status deployment/blogginrose-ui --timeout=60s
                            kubectl get pods -l app=blogginrose-ui
                        '
                      '''
                  }
              }
          }
      }

      post {
          success { echo 'Deployment successful!' }
          failure { echo 'Deployment failed!' }
      }
  }

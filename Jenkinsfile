pipeline {
     agent any
     stages {
         stage('Build') {
              steps {
                  sh 'echo Building...'
              }
         }
          
         
         stage('Lint HTML') {
              steps {
                  sh 'tidy -q -e *.html'
              }
         }
         stage('Build Docker Image') {
              steps {
                  sh 'docker build -t capstone-project .'
              }
         }
         stage('Push Image To Dockerhub') {
              steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                  sh '''
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push hballow/capstone-project
                  '''
                }
              }
         }
         stage('Deploying') {
              steps{
                  echo 'Deploying to AWS...'
                  withAWS(credentials: 'aws', region: 'us-east-2') {
                      sh "aws eks --region us-east-2 update-kubeconfig --name capstone_cluster"
                      sh "kubectl config use-context arn:aws:eks:us-east-2:725326878966:cluster/capstone_cluster"
                      sh "kubectl set image deployments/capstone-project-cloud-devops capstone-project-cloud-devops=sabbir33/capstone-project-cloud-devops:latest"
                      sh "kubectl apply -f deployment/deployment.yml"
                      sh "kubectl get nodes"
                      sh "kubectl get deployment"
                      sh "kubectl get pod -o wide"
                      sh "kubectl get service/capstone-project"
                  }
              }
        }
        stage("Cleaning up") {
              steps{
                    echo 'Cleaning up...'
                    sh "docker system prune"
              }
        }
     }
}

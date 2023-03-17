pipeline {
    agent { label 'master' }
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "1devika/train-schedule"
        JAVA_HOME = "/usr/lib/jvm/java-8-oracle/"
        KUBECONFIG = "/var/lib/jenkins/devops-cluster-admin-config"

    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                      sh "docker build -t $DOCKER_IMAGE_NAME ."
                    }
                }
            }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker_hub_login', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                      sh '''
                        docker login -u="${DOCKER_USERNAME}" -p="${DOCKER_PASSWORD}"
                        docker push $DOCKER_IMAGE_NAME
                      '''
                      }
                }
            }
        }
        stage('CanaryDeploy') {
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                sh "kubectl apply -f train-schedule-kube-canary.yml"
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                // kubernetesDeploy(
                //     kubeconfigId: 'kubeconfig',
                //     configs: 'train-schedule-kube-canary.yml',
                //     enableConfigSubstitution: true
                // )
                // kubernetesDeploy(
                //     kubeconfigId: 'kubeconfig',
                //     configs: 'train-schedule-kube.yml',
                //     enableConfigSubstitution: true
                // )
    
                sh "kubectl scale deployment --replicas 0 train-schedule-deployment-canary "
                sh "kubectl apply -f train-schedule-kube.yml"
            }
        }
    }
}

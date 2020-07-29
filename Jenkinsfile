pipeline {
    environment {
        registry = "taha3azab/webapi-demo"
        registryCredential = 'dockerhub-credentials'
    }
    agent any
    stages {
        stage('Hashing images') {
            steps {
                script {
                    env.GIT_HASH = sh(
                        script: "git show --oneline | head -1 | cut -d' ' -f1",
                        returnStdout: true
                    ).trim()
                }
            }
        }
        stage('Lint Dockerfile') {
            agent {
                docker {
                    image 'hadolint/hadolint:latest-debian'
                }
            }
            post {
                always {
                    archiveArtifacts 'hadolint_lint.txt'
                }
            }
            steps {
                sh 'hadolint ./Dockerfile | tee -a hadolint_lint.txt'
            }
        }
        stage('Build & Push to dockerhub') {
            steps {
                script {
                    dockerImage = docker.build(registry + ":${env.GIT_HASH}")
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                sh "docker run --name webapi-demo -d -p 80:80 $registry:${env.GIT_HASH}"
            }
        }        
        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi $registry:${env.GIT_HASH}"
            }
        }
        stage('Cleaning Docker') {
            steps {
                script {
                    sh "echo 'Cleaning Docker'"
                    sh "docker stop webapi-demo"
                    sh "docker system prune -f"
                }
            }
        }
        // stage('Deploy to EKS') {
        //     steps {
        //         echo 'Deploying to AWS...'
        //         dir('kubernetes') {
        //             withAWS(region: awsRegion, credentials: awsCredentials) {
        //                 sh "aws eks --region $awsRegion update-kubeconfig --name UdacityCapstoneProject-Cluster"
        //                 sh "kubectl config use-context arn:aws:eks:us-east-1:957920350523:cluster/UdacityCapstoneProject-Cluster"
        //                 sh "kubectl apply -f deployment.yml"
        //                 sh "kubectl get nodes"
        //                 sh "kubectl get deployment"
        //                 sh "kubectl get pod -o wide"
        //                 sh "kubectl get service/capstone-app"
        //             }
        //         }
        //     }
        // }
    }
}
pipeline {
    agent { label 'master' }

    environment {
        REGION="us-east-1"
        ECR_REGISTRY= "375516740190.dkr.ecr.us-east-1.amazonaws.com"
        REPO_NAME="bb4it/pb-jenkins"
    }
    stages { 
        stage ('Create ECR repository') {
            steps {
                echo 'Creating Repository in AWS ECR'
                sh """
                aws ecr create-repository \
                        --repository-name ${REPO_NAME} \
                        --image-scanning-configuration scanOnPush=false \
                        --image-tag-mutability MUTABLE \
                        --region ${REGION}
                        """
            }
        }

        stage ('Docker build'){
            steps {
                echo ' Building Docker image '
                sh 'docker build --force-rm -t "${ECR_REGISTRY}/${REPO_NAME}:latest" .'
            }
        }
        stage ('ECR login') {
            steps {
                echo ' Login into AWS ECR '
                sh """
                aws ecr get-login-password \
                --region ${REGION} \
                | docker login \
                --username AWS \
                --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage ('Push Docker image to ECR'){
            steps {
                echo 'Pushing Docker image to AWS ECR '
                sh 'docker push "${ECR_REGISTRY}/${REPO_NAME}:latest"'
            }
        }
        
        stage ('Creating infrastructure') {
            steps { 
            echo ' Creating infrastructure for the app'
            sh 'aws cloudformation create-stack --stack-name project204 --template-body file://project-204-cfn-template.yml --parameters ParameterKey=KeyPairName,ParameterValue=betul --region us-east-1 --capabilities CAPABILITY_IAM '
            }
        }

        stage ('Testing infrastructure') {
            steps { 
            echo ' Testing Docker Swarm readiness'
            
            }
        }

        stage ('Deploy the App') {
            steps { 
            echo ' Deploying the App on Docker Swarm'
            sh 'ls -la'
            sh 'aws cloudformation create-stack --stack-name project204 --template-body file:///var/lib/jenkins/workspace/project-204/Jenkins-Pipeline-for-Dockerized-Phonebook-Application/project204-cfn-template.yml --parameters ParameterKey=KeyPairName,ParameterValue=betul --region us-east-1 --capabilities CAPABILITY_IAM '
            }
        }

        stage ('Test the App') {
            steps { 
            echo ' Checkin App readiness'
            sh 'ls -la'
            sh 'aws cloudformation create-stack --stack-name project204 --template-body file:///var/lib/jenkins/workspace/project-204/Jenkins-Pipeline-for-Dockerized-Phonebook-Application/project204-cfn-template.yml --parameters ParameterKey=KeyPairName,ParameterValue=betul --region us-east-1 --capabilities CAPABILITY_IAM '
            }
        }
            //script {
                //sh ' aws ec2 describe-instances --region us-east-1 \
                //--filters "Name=tag:Name, Values=GrandMaster" \
                //--output=text
            //export PUBLIC_IP= $(mssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -r us-east-1 ${DockerManager1}  curl http://169.254.169.254/latest/meta-data/public-ipv4)


            //}
        post {
            always {
                echo 'Deleting all local images'
                sh ' docker image prune -af '
            }
            failure {
                echo 'Delete image repository from ECR on failure'
                echo 'Delete Cloudformation Stack on failure'

            }
        }    

    }
        
}       
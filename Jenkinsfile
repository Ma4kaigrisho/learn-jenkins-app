pipeline {
    agent any
    environment{
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'jenkinsapp'
        AWS_DEFAULT_REGION = 'eu-north-1'
        AWS_ECS_CLUSTER = "JenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE_PROD = "JenkinsApp-Service-Prod"
        AWS_ECS_TD_PROD = 'JenkinsApp-TaskDefinition-Prod'
        AWS_DOCKER_REGISTRY='693744224674.dkr.ecr.eu-north-1.amazonaws.com'
    }
    stages {
        stage('Build') {
            agent{
                docker{
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage("Build Docker image"){
            agent{
                docker{
                    image 'my-aws-cli'
                    args "-u root --entrypoint='' -v /var/run/docker.sock:/var/run/docker.sock"
                    reuseNode true
                }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }
        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'my-aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }
            }
            environment{
                BUCKET_URL = 'learn-jenkins-202503181115'
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        sed -i "s/#APP-VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                        LATEST_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq .'taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }       
    }
    
}

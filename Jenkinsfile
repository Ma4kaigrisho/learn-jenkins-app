pipeline {
    agent any
    environment{
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'eu-north-1'
    }
    stages {
        stage('Deploy to AWS'){
            agent{
                docker{
                    image 'amazon/aws-cli'
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
                        yum install jq -y
                        LATEST_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq .'taskDefinition.revision')
                        echo $LATEST_REVISION
                        aws ecs update-service --cluster JenkinsApp-Cluster-Prod --service JenkinsApp-Service-Prod --task-definition JenkinsApp-TaskDefinition-Prod:$LATEST_REVISION
                        aws ecs wait services-stable --cluster JenkinsApp-Cluster-Prod --services JenkinsApp-Service-Prod
                    '''
                }
            }
        }       
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
 
    }
    
}

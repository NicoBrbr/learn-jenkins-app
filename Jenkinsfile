pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'eu-west-3'
    }

    stages {
        stage('Deploy to AWS'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision') 
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster arn:aws:ecs:eu-west-3:072307788259:cluster/LearnJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Service-Prod --task-definition arn:aws:ecs:eu-west-3:072307788259:task-definition/LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_REVISION
                        aws ecs wait service-stable --cluster arn:aws:ecs:eu-west-3:072307788259:cluster/LearnJenkinsApp-Cluster-Prod --services LearnJenkinsApp-Service-Prod
                    '''
                }
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'small changes'
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

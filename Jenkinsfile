def buildNumber = Jenkins.instance.getItem('cicd-jenkins-bean-stage').lastSuccessfulBuild.number

def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline {
    agent any

    environment {
        ARTIFACT_NAME="vprofile-v${buildNumber}.war"
        AWS_S3_BUCKET="elasticbeanstalk-us-east-1-930052591067"
        AWS_EB_APP_NAME="vproapp"
        AWS_EB_ENVIRONMENT="Vproapp-PROD-env"
        AWS_EB_APP_VERSION="${buildNumber}"
    }

    stages {
        stage('Deploy to AWS Beanstalk Prod env'){
            steps {
                withAWS(credentials: 'ebsDeployment', region: 'us-east-1') {
                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
            }
        }
    }

    post{
        always {
            echo 'Sending result to Slack'
            slackSend channel: '#jenkinscicd',
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}

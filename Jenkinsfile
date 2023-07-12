def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger'
]

pipeline {
    agent any
    tools {
        maven "maven3"
        jdk "OpenJdk11"
    }

    environment {
        SNAP_REPO='vprofile-snapshot'
        NEXUS_USER='admin'
        RELEASE_REPO='vprofile-release'
        CENTRAL_REPO='vpro-maven-central'
        NEXUSIP='172.31.27.240' // Private IP of Nexus Host 
        NEXUSPORT='8081'
        NEXUS_GRP_REPO='vpro-maven-group'
        NEXUS_LOGIN='nexus-login'
        SONARSERVER='sonarserver'
        SONARSCANNER='sonarscanner'
        NEXUSPASS=credentials('nexuspass')
        ARTIFACT_NAME="vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET="elasticbeanstalk-us-east-1-930052591067"
        AWS_EB_APP_NAME="vproapp"
        AWS_EB_ENVIRONMENT="Vproapp-STAGING-env"
        AWS_EB_APP_VERSION="${BUILD_ID}"
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        // stage('Sonar Analysis'){
        //     environment {
        //         scannerHome = tool "${SONARSCANNER}"
        //     }
        //     steps {
        //         withSonarQubeEnv("${SONARSERVER}") {
        //             sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
        //             -Dsonar.projectName=vprofile \
        //             -Dsonar.projectVersion=1.0 \
        //             -Dsonar.sources=src/ \
        //             -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
        //             -Dsonar.junit.reportsPath=target/surefire-reports/ \
        //             -Dsonar.jacoco.reportsPath=target/jacoco.exec \
        //             -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
        //             '''
        //         }
        //     }
        // }

        // stage("Quality Gate") {
        //     steps {
        //         timeout(time: 30, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        // stage("UploadArtifact"){
        //     steps{
        //         nexusArtifactUploader(
        //             nexusVersion: 'nexus3',
        //             protocol: 'http',
        //             nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
        //             groupId: 'QA',
        //             version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        //             repository: "${RELEASE_REPO}",
        //             credentialsId: "${NEXUS_LOGIN}",
        //             artifacts: [
        //                 [
        //                     artifactId: 'vproapp',
        //                     classifier: '',
        //                     file: 'target/vprofile-v2.war',
        //                     type: 'war'
        //                 ]
        //             ]
        //         )
        //     }
        // }

        stage('Deploy to AWS Beanstalk Staging env'){
            steps {
                withAWS(credentials: 'ebsDeployment', region: 'us-east-1') {
                    sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                    sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
            }
        }

    //     stage('TODO: need to deploy the file as ROOT.war'){}

        // stage('Deploy to BeanStalk') {
        //     steps {
        //         step(
        //             [
        //                 $class: "AWSEBDeploymentBuilder", 
        //                 zeroDowntime: false,
        //                 awsRegion: "us-east-1",
        //                 applicationName: "${AWS_EB_APP_NAME}",
        //                 environmentName: "${AWS_EB_ENVIRONMENT}",
        //                 bucketName: "${AWS_S3_BUCKET}",
        //                 rootObject: "target",
        //                 includes: "vprofile-v2.war",
        //                 credentialId: "ebsDeployment",
        //                 versionLabelFormat: "${AWS_EB_APP_VERSION}", 
        //                 versionDescriptionFormat: ""
        //             ]
        //         )
        //     } 
        // }
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

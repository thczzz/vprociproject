pipeline {
    agent any
    tools {
        maven "maven3"
        jdk "OpenJdk11"
    }

    environment {
        SNAP_REPO='vprofile-snapshot'
        NEXUS_USER='admin'
        NEXUS_PASS='admin123'
        RELEASE_REPO='vprofile-release'
        CENTRAL_REPO='vpro-maven-central'
        NEXUSIP='172.31.27.240' // Private IP of Nexus Host 
        NEXUSPORT='8081'
        NEXUS_GRP_REPO='vpro-maven-group'
        NEXUS_LOGIN='nexus-login'
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
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}

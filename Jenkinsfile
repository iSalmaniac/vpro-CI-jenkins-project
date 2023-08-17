def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    agent any

	tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }		
    environment {
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.14.21'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER= 'sonarserver'
        SONARSCANNER= 'sonarscanner'
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET = "vpro-cicd-beanstalk"
        AWS_EB_APP_NAME = "vpro-app-cicd-beanstalk"
        AWS_EB_ENVIRONMENT = "Vpro-app-cicd-beanstalk-env" 
        AWS_EB_APP_VERSION = "${BUILD_ID}"
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
        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                  waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUS_PORT}",
                    groupID: 'QA',
                    verson: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                         classifier: '',
                         file: 'target/vprofile-v2.war',
                         type:'war']
                    ]
                )
            }
        }
                



    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#devopscicdproject',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"


        }
    }
}

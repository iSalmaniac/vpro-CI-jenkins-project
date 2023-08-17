def buildNumber = Jenkins.instance.getItem('vprofile-CICD-jenkins-bean-stage').lastSuccessfulBuild.number

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
	
        SONARSERVER= 'sonarserver'
        SONARSCANNER= 'sonarscanner'
        ARTIFACT_NAME = "vprofile-v${buildNumber}.war"
        AWS_S3_BUCKET = "vpro-cicd-beanstalk"
        AWS_EB_APP_NAME = "vpro-app-cicd-beanstalk"
        AWS_EB_ENVIRONMENT = "Vproapp-prod-environment-env" 
        AWS_EB_APP_VERSION = "${buildNumber}"

    }
   
        stage("Deploy to Stage Beanstalk"){
          steps {
            withAWS(credentials: 'awsbeancreds', region: 'ap-south-1') {
               sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
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

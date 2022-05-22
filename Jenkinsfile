pipeline {
    agent any
	parameters {
	booleanParam(name: 'PROMOTE',  description: 'If toogled new version should be published')
	
	text(name: 'VERSION', defaultValue: '', description: 'Version number that will be published')
	}        
    stages {		
	stage('Publish') {
            when {
                expression { params.PROMOTE == true }
             }
              steps {
		echo '.::Publishing::.'	 
                sh 'ssh deployer@192.168.0.107 rm -rf ~/repo/'
                sh 'ssh deployer@192.168.0.107 mkdir ~/repo'
                sh 'scp output_volume/final_app.jar deployer@192.168.0.107:~/repo/'
                sh """mv output_volume/final_app.jar output_volume/'app_realease-${params.VERSION}.jar'"""
                archiveArtifacts artifacts: """output_volume/app_realease-${params.VERSION}.jar""", fingerprint: true			
            }
        }
    }
    post {
	 always {
            cleanWs()
        }
        success {
            echo '.::Succeeded, Saving artifact::.'
        }
        failure{
                echo '.::Pipeline FAILED::.'
                mail bcc: '', body: "<b>Pipeline FAILED</b><br>\n<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Jenkins URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '',subject: "ERROR CI: in project -> ${env.JOB_NAME}", to: "mich.pieczonka@gmail.com";
        }
    }
}

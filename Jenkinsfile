pipeline {
    agent any
    stages {
          stage('Deploy') {
            steps {
                script {                              
					timeout (1) {
						try {
							echo 'building deploy image'
							sh 'docker build . --no-cache -f Dockerpublish -t deploy'
							echo 'starting application in pure dev container'
							sh 'docker run --name deploy-container -d -p 8989:80 deploy'
							sh 'sleep 20'
							final String url = "http://localhost:8080"
                            final String response = sh(script: "curl -s $url", returnStdout: true).trim()
							
							sh 'sleep 15000'
						} catch (Exception e) {
							echo 'Exception during timeout has been thrown with message'
                            echo e.toString()
                            if (e.toString() == "org.jenkinsci.plugins.workflow.steps.FlowInterruptedException") {
                                sh 'docker stop deploy-container'
                                sh 'docker rm deploy-container'
                                echo 'Deployed successfully!'
                            } else {
                                throw new Exception(e.toString())
                        }                   
				echo 'I made it here after catching exception'
				}
            }
        }
    }
}
    }
    post {
        success {
            echo 'Succeeded, now I`m saving artifact.'
            archiveArtifacts artifacts: 'shared_volume/app.jar', fingerprint: true
        }
        failure {
            echo 'Failed, I`m not saving any artifacts.'
        }
    }
}

pipeline {
    agent any
    stages {

        
        stage('Deploy') {
            steps {
                sh 'docker.build("petclinic", ". --no-cache -f Dockerpublish")'
                sh 'docker run -d -p 8989:80 petclinic'
                script {                              
                      try {
                          timeout(time: 1, unit: 'MINUTES') {
                           }
                        } catch (Exception e) {
                            echo e.toString()
                            if (e.toString() == "org.jenkinsci.plugins.workflow.steps.FlowInterruptedException") {
                                sh 'docker stop -f petclinic'
                                echo 'Deployed successfully!'
                            } else {
                                throw new Exception(e.toString())
                            }
                        }
                   sh 'docker stop petclinic' 
                   sh 'docker rm -f petclinic'
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

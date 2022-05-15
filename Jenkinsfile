pipeline {
    agent any
    stages {

        stage('Deploy') {
            steps {
                script {                              
                      try {
                          def deployImage = docker.build("petclinic", ". --no-cache -f Dockerpublish")
                          deployImage.inside{
                              timeout(1){}
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

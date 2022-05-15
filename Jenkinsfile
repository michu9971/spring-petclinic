pipeline {
    agent any
    stages {

          stage('Deploy') {
            steps {
                script {                              
 timeout (1) {
    try {
      sh 'sleep 15000'
    } catch (Exception ex) {
      echo 'In exception'
    }  
    echo 'I made it here after catching exception'
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

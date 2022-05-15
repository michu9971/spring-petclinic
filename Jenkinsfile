pipeline {
    agent any
    stages {
        stage("Pull dependencies") {
            steps {
                script {
                    docker.build("predependencies", ". -f Dockerdep")
                    sh 'echo PreDependencies container has been built'
                    sh 'docker run -v \$(pwd)/maven-dependencies:/root/.m2 -w /petclinic-app --name temp-container predependencies mvn dependency:resolve'
                    sh 'docker commit temp-container dependencies'
                    sh 'docker rm temp-container'
                }
            }
        }
        

        stage('Deploy') {
            steps {
                script {
                      try {
                          def deployImage = docker.build("petclinic", ". --no-cache -f Dockerpublish")
                          deployImage.run("--name petclinic -d -p 8989:80")
                          timeout(time: 1, unit: 'MINUTES') {
                          sh 'ls -l'
                          sh 'echo  $(jestem w srodku)'
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

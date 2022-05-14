pipeline {
    agent any
    environment {
    DOCKER_BUILDKIT='1'
}
    stages {
        stage("Pull dependencies") {
            steps {
                script {
                    docker.build("predependencies", ". --no-cache -f Dockerdep")
                    sh 'echo PreDependencies container has been built'
                    sh 'docker run -v \$(pwd)/maven-dependencies:/root/.m2 -w /petclinic-app --name temp-container predependencies mvn dependency:go-offline'
                    sh 'docker commit --change="CMD bash" temp-container dependencies'
                    sh 'docker rm temp-container'
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh 'ls'
                    def imageBuild = docker.build("builder", ". --no-cache -f Dockerbuild")
                    sh 'rm -rf output_volume'
                    sh 'mkdir output_volume'
                    imageBuild.run("-v \$(pwd)/output_volume:/build_result")
                    sh 'ls shared_volume'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.build("tester", ". -f Dockertest")
                    sh 'echo tested'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {           
                     def deployImage = docker.build("petclinic", ". -f Dockerpublish")
                      try {
                            timeout(time: 1, unit: 'MINUTES') {
                              deployImage.run("--name petclinic")
                            }
                        } catch (Exception e) {
                            echo e.toString()
                            if (e.toString() == "org.jenkinsci.plugins.workflow.steps.FlowInterruptedException") {
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

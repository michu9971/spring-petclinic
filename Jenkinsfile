pipeline {
    agent any
    stages {
        stage("Pull dependencies") {
            steps {
                script {
                    sh 'ls'
                    docker.build("petclinic-dep", ". --no-cache -f Dockerdep")
                    sh 'docker run --rm -v \$(pwd)/maven-dependencies:/root/.m2 -w /petclinic-app petclinic-dep mvn --version'
                         
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    sh 'ls'
                    def imageBuild = docker.build("petclinic-build", ". --no-cache -f Dockerbuild")
                    sh 'rm -rf shared_volume'
                    sh 'mkdir shared_volume'
                    imageBuild.run("-v \$(pwd)/shared_volume:/output")
                    sh 'ls shared_volume'
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    docker.build("petclinic-test", ". -f Dockertest")
                    sh 'echo tested'
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh 'docker rm -f petclinic'
                    def deployImage = docker.build("petclinic", ". -f Dockerpublish")
                    deployImage.run("--name petclinic")
                    sh 'sleep 5'
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

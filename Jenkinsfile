def CONTAINER_ID
pipeline {
    agent any
    environment {
    DOCKER_BUILDKIT='1'
}
    stages {
        stage("Pull dependencies") {
            steps {
                script {
                    sh 'ls'
                    docker.build("dependencies", ". -f Dockerdep")
                    sh 'echo Dependencies container has been built' 
                    sh 'CONTAINER_ID=$(docker run -dit -v \$(pwd)/maven-dependencies:/root/.m2 dependencies)'
                    sh """docker exec -t dependencies -c 'cd /petclinic-app mvn clean'"""
                    sh 'docker commit dependencies dependencies:latest'
                    sh 'docker stop dependencies'

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

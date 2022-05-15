pipeline {
    agent any
	parameters {
	booleanParam(name: 'PROMOTE',  description: 'If toogled new version should be published')
	
	text(name: 'VERSION', defaultValue: '', description: 'Version number that will be published')
	}        
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
        
        stage('Build') {
            steps {
                script {
                    sh 'ls'
                    def imageBuild = docker.build("builder", ". -f Dockerbuild")
                    sh 'rm -rf output_volume'
                    sh 'mkdir output_volume'
                    imageBuild.run("-v \$(pwd)/output_volume:/build_result")
                    sh 'ls output_volume'
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
			timeout (1) {
				try {
				     echo 'building deploy image'
				     sh 'docker build . --no-cache -f Dockerpublish -t deploy'
				     echo 'starting application in pure dev container'
				     sh 'docker run --name deploy-container -d -p 8989:80 deploy'
				     sh 'sleep 20'
				     final String url = "http://localhost:8080"
                                     final String response = sh(script: "curl -s $url", returnStdout: true).trim()
                                     echo response							
				     sh 'sleep 150'
				} catch (Exception e) {
				     echo 'Exception during timeout has been thrown with message'
                                     echo e.toString()
					if (e.toString() == "org.jenkinsci.plugins.workflow.steps.FlowInterruptedException") {
						echo 'Deployed successfully!'
					} else {
						throw new Exception(e.toString())
					} 
                               sh 'docker stop deploy-container'
                               sh 'docker rm deploy-container'                                                   
						}
					}
				}
			}
		}		
	stage('Publish') {
            when {
                expression { params.PROMOTE == true }
             }
              steps {
			    echo '.::Publishing::.'	
				script {
                                        
				def sourceJarFile = "output_volume/final_app.jar"                                  
                                 if (fileExists(file: sourceJarFile)) {
                                       def artifactName = """'app_realease-${params.VERSION}.jar'"""                                     
                                        writeFile(file: artifactName, encoding: "UTF-8", text: readFile(file: sourceFile, encoding: "UTF-8"))
                                        archiveArtifacts artifacts: finalArctifact, fingerprint: true
                                 }
			     
				}				
            }
        }
    }
    post {
        success {
            echo 'Succeeded, now I`m saving artifact.'
        }
        failure {
            echo 'Failed, I`m not saving any artifacts.'
        }
    }
}

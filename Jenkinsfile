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
				script {
                                        
				def sourceJarFile = "output_volume/final_app.jar"                                  
                                 if (fileExists(file: sourceJarFile)) {
                                       def artifactName = """'app_realease-${params.VERSION}.jar'"""                                     
                                        writeFile(file: artifactName, encoding: "UTF-8", text: readFile(file: sourceJarFile, encoding: "UTF-8"))
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

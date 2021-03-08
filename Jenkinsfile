pipeline{
	agent any
	tools{
		maven 'Maven'
	}
	stages {
		stage ('Initialize'){
			steps {
        			sh '''
                    			echo "PATH = ${PATH}"
                    			echo "M2_HOME = ${M2_HOME}"
            			''' 
      			}
		}
		
    		stage ('Secret Check') {
      			steps {
        			sh 'rm trufflehog || true'
        			sh 'docker run gesellix/trufflehog --json https://github.com/tewan-dtt/webapp.git  > trufflehog'
        			sh 'cat trufflehog'
      			}
    		}
		
		stage('Fortify Remote Analysis (SAST)') {
      			steps {
        			fortifyRemoteArguments transOptions: '-Xmx2G'
				fortifyRemoteAnalysis remoteAnalysisProjectType: fortifyMaven(), uploadSSC: [appName: 'Jenkins-poc', appVersion: '1']
      			}
    		}
		
		stage ('Building Artifact') {
      			steps {
      				sh 'mvn clean package'
       			}
    		}
		
		stage ('Container Baking'){
			environment {
				registry = "wanhyterr/webapp"
				registryCredential = 'docker-hub'
			}
			steps{
				script{
					def webapp = docker.build("webapp")
					webapp.push('latest')
				}
			}
		}
		
		stage ('Deployment to Staging'){
			steps{
				sshagent(['tomcat']){
					sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@tomcat:/usr/share/tomcat/webapps/webapps.war'
				}
			}
		}
		
		stage ('Fortify WebInspect (DAST)'){
			steps {
				sh 'pwsh /opt/webinspect_script/webinspect-webapps.ps1'
			}
		}
	}
}

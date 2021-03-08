pipeline{
	agent { dockerfile true }
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
		
		stage ('Check-Git-Secrets') {
      			steps {
        			sh 'rm trufflehog || true'
        			sh 'docker run gesellix/trufflehog --json https://github.com/tewan-dtt/webapp.git  > trufflehog'
        			sh 'cat trufflehog'
      			}
    		}
		
		stage('Fortify Remote Arguments') {
      			steps {
        			fortifyRemoteArguments transOptions: '-Xmx2G'
      			} 
    		} 
    
		stage('Fortify Remote Analysis') {
      			steps {
        			fortifyRemoteAnalysis remoteAnalysisProjectType: fortifyMaven(), uploadSSC: [appName: 'Jenkins-poc', appVersion: '1']
      			}
    		}
		
		stage ('Build') {
      			steps {
      				sh 'mvn clean package'
       			}
    		}
		
		stage ('Container Baking'){
			steps{
				def webapp = docker.build("webapp","./src/main")
				webapp.push('latest')
			}
		}
		
		stage ('Deploy-To-Tomcat'){
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

pipeline{
	agent any
	environment{
		dockerImage = ''	
	}
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
		
		stage ('Container Image Baking'){
			steps{
				script{
					dockerImage = docker.build("webapp")
				}
			}
		}
		
		stage ('Container Image Upload'){
			steps{
				docker.withRegistry('wanhyterr/webapp','docker-hub')
				script{
					dockerImage.push('latest')
				}
			}
		}
		
		stage ('Container Image Run'){
			steps{
				sh 'docker ps -f name=webapp -q | xargs --no-run-if-empty docker container stop'
				sh 'docker container ls -a -fname=webapp -q | xargs -r docker container rm'
				script{
					dockerImage.run("-p 8888:8080 --rm -name webapp")
				}
			}
		}
		
		stage ('Fortify WebInspect (DAST)'){
			steps {
				sh 'pwsh /opt/webinspect_script/webinspect-webapps.ps1'
			}
		}
		
		stage ('WAR Artifact Transfer'){
			steps{
				sshagent(['tomcat']){
					sh 'scp -o StrictHostKeyChecking=no target/*.war ec2-user@tomcat:/usr/share/tomcat/webapps/webapps.war'
				}
			}
		}
	}
}

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
		
		stage ('Artifact and Image Upload'){
			environment {registryCredential = 'docker-hub'}
			steps{
				script{
					sh 'cp target/webapps.war /tmp/webapps-${BUILD_NUMBER}.war'
					docker.withRegistry( '', registryCredential ) {
						dockerImage.push("$BUILD_NUMBER")
						dockerImage.push('latest')
					}
				}
			}
		}
		
		stage('Prisma Cloud Scan') {
            		steps {
				// Scan the image
				prismaCloudScanImage ca: '',
				cert: '',
				dockerAddress: 'unix:///var/run/docker.sock',
				image: 'webapp*',
				key: '',
				logLevel: 'info',
				podmanPath: '',
				project: '',
				resultsFile: 'prisma-cloud-scan-results.json',
				ignoreImageBuildTime:true
            		}
        	}
		
		stage ('Deploy in Staging'){
			steps{
				sh 'docker ps -f name=webapp -q | xargs --no-run-if-empty docker container stop'
				sh 'docker container ls -a -fname=webapp -q | xargs -r docker container rm'
				script{
					dockerImage.run("-p 8888:8080 --rm --name webapp")
				}
			}
		}
		
		stage ('Fortify WebInspect (DAST)'){
			steps {
				sh 'pwsh /opt/webinspect_script/webinspect-webapps.ps1'
			}
		}
		
		stage ('Approve to Deploy in Prod'){
			steps{
				input "Are you sure to deploy the new release into production? Please ensure the new release does not contain any severe vulnerabilities."
			}
		}
		
	}
}

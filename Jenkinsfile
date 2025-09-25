pipeline{
	agent any
	environment {
		PATH = "/opt/maven/bin:$PATH"
	}
	stages {

		stage('Build') {
			steps {
				sh 'mvn clean deploy'
				}
			}
		stage('SonarQube analysis'){
			environment {
				scannerHome = tool 'piyush-sonar-scanner'
			}
		
			steps {
				withSonarQubeEnv('piyush-sonar-server') {
					sh "${scannerHome}/bin/sonar-scanner"
				}
			
			}
		}
	}
}

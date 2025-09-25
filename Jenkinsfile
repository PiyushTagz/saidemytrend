// Define the URL of the Artifactory registry
def registry = 'https://piyush05.jfrog.io'

pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {
        stage('Build') {
            steps {
                echo "..............Build started............"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "................Build completed........"
            }
        }

        stage('Test') {
            steps {
                echo "..............Unit test started............"
                sh 'mvn surefire-report:report'
                echo "................Unit test completed........"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'piyush-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('piyush-sonar-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            echo "WARNING: Quality Gate failed but continuing pipeline. Status: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Jar Publication') {
            steps {
                script {
                    echo '..................Jar Publication Started........'

                    // Define Artifactory server
                    def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifact-cred"

                    // Define properties
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"

                    // Upload specification
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "piyush-libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""

                    // Upload and publish build info
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)

                    echo '<------------- Jar Publish Ended ------------->'
                }
            }
        }
    }
}


def COLOR_MAP = [
    'SUCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
        maven "MAVEN3.9.9"
        jdk "JDK21"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'admin123'
		RELEASE_REPO = 'vprofile-release'
		CENTRAL_REPO = 'vpro-maven-central'
		NEXUSIP = '172.31.15.94'
		NEXUSPORT = '8081'
		NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving3."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh """${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                      -Dsonar.projectName=vprofile \
                      -Dsonar.projectVersion=1.0 \
                      -Dsonar.sources=src/ \
                      -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                      -Dsonar.junit.reportsPath=target/surefire-reports/ \
                      -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                      -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml"""
                }
            }

        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    //parameter indicates wether to set pipeline UNSTABLE
                    //true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Upload to Nexus") {
           steps {
               nexusArtifactUploader(               // ✅ Must use opening parenthesis (
                     nexusVersion: 'nexus3',
                      protocol: 'http',
                      nexusUrl: '172.31.15.94:8081',
                      groupId: 'com.visualpathit',
                      version: "${env.BUILD_ID}",
                      repository: 'vprofile-release',
                      credentialsId: 'nexuslogin',
                      artifacts: [
                        [
                           artifactId: 'vprofile-v2',
                           classifier: '',
                           file: 'target/vprofile-v2.war',
                           type: 'war'
                    ]
                 ]
              )
           }
        }
    }
    post {
        always {
            echo 'Slack Notification.'
            slackSend channel: '#jenkinscic',
            color: COLOR_MAP[currentBuild.currentResult],
            slackSend (channel: '#jenkinscic', message: "Pipeline failed: ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info: ${env.BUILD_URL}")
        }
    }
}               


         
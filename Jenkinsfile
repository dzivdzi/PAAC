def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages {
    stage('Fetch code'){
      steps {
        git branch: 'your-git-branch-here', url: 'your-git-url-here'
      }
    }


    stage('Build'){
      steps {
        sh 'mvn install'
      }

      post {
        success{
            echo 'Now archiving...'
            archiveArtifacts artifacts: '**/target/*.war'
        }
      }
    }

    stage ('Test') {
        steps {
            sh 'mvn test'
        }
    }

    stage ('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }



    stage('build && SonarQube analysis') {
        environment {
            scannerHome = tool 'you-jenkins-name-for-sonar-here'
        }
        steps {
            withSonarQubeEnv('sonar') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                -Dsonar.projectName=vprofile-repo \
                -Dsonar.projectVersion=1.0 \
                -Dsonar.sources=src/ \
                -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
        }
    }

    stage("Quality Gate") {
        steps {
            timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
                waitForQualityGate abortPipeline: false
            }
        }
    }

    stage("Upload Artifact"){
        steps{
        nexusArtifactUploader(
        nexusVersion: 'nexus3',
        protocol: 'http',
        nexusUrl: 'your-nexus-url-here',
        groupId: 'your-group-id-here',
        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        repository: 'your-repo-name-here',
        credentialsId: 'your-nexus-token-for-jenkins-here',
        artifacts: [
            [artifactId: 'artifact-name-here',
                classifier: '',
                file: 'path/to/artifact/here',
                type: 'file-tipe-of-artifact-here']
            ]
        )
        }
    }

    post{
        always{
            echo 'Slack Notifications'
            slackSend channel: '#you-slack-channel-name-here',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }

  }
}
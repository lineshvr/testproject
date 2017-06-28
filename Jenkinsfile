pipeline {
    agent any
    stages {
        stage('Branch build') {
            agent any
            steps {
                script {
                    env.commitId = readCommitId()
                    env.commitMessage = readCommitMessage()
                    env.version = DateTimeFormatter.ofPattern('yyyy-MM-dd-HHmm').format(now(ZoneId.of('UTC'))) + "-" + env.commitId.take(6)
                    if (isQuickBuild()) {
                        currentBuild.description = "Building: ${env.commitId}"
                        bat 'mvn clean verify'
                    }
                }
            }
        }

	  stage('Release build') {
            when { branch 'master' }
            agent any
            steps {
                script {
                    currentBuild.description = "Release: ${env.version}"
                    bat "mvn versions:set -DnewVersion=${env.version}"
                    bat 'mvn clean deploy -B'
                    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                    tagPuppetModules("${env.version}")
                }
            }
        }
	}
		}
		
		
		String readCommitId() {
    return bat(returnStdout: true, script: 'git rev-parse HEAD').trim()
}

String readCommitMessage() {
    return bat(returnStdout: true, script: 'git log -1 --pretty=%B').trim()
}


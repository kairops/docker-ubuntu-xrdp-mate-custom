#!groovy

@Library('github.com/teecke/jenkins-pipeline-library@v3.4.1') _

// Initialize global config
cfg = jplConfig('duing', 'docker', '', [email: env.CIKAIROS_NOTIFY_EMAIL_TARGETS])

/**
 * Build and publish docker images
 *
 * @param nextReleaseNumber String Release number to be used as tag
 */
def buildAndPublishDockerImage(nextReleaseNumber = "") {
    if (nextReleaseNumber == "") {
        nextReleaseNumber = sh (script: "kd get-next-release-number .", returnStdout: true).trim().substring(1)
    }
    docker.withRegistry("", 'cikairos-docker-credentials') {
        def customImage = docker.build("kairops/duing:${nextReleaseNumber}", "--pull --no-cache duing")
        customImage.push()
        if (nextReleaseNumber != "beta") {
            customImage.push('latest')
            customImage.push('19.04')
        }
    }
}

pipeline {
    agent { label 'docker' }

    stages {
        stage ('Initialize') {
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Build') {
            steps {
                buildAndPublishDockerImage("beta")
            }
        }
        stage ('Make release') {
            when { branch 'release/new' }
            steps {
                buildAndPublishDockerImage()
                jplMakeRelease(cfg, true)
            }
        }
    }

    post {
        always {
            jplPostBuild(cfg)
        }
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(artifactNumToKeepStr: '20',artifactDaysToKeepStr: '30'))
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
    }
}

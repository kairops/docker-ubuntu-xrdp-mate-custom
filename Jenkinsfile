#!groovy

@Library('github.com/red-panda-ci/jenkins-pipeline-library@v3.1.5') _

// Initialize global config
cfg = jplConfig('duing', 'docker', '', [email:'redpandaci+duing@gmail.com'])

pipeline {
    agent none

    stages {
        stage ('Initialize') {
            agent { label 'docker' }
            steps  {
                jplStart(cfg)
            }
        }
        stage ('Build') {
            agent { label 'docker' }
            steps {
                script {
                    docker.build('kairops/duing:test', '--pull --no-cache ./duing')
                }
            }
        }
        stage ('Test') {
            agent { label 'docker' }
            steps  {
                sh 'bin/test.sh'
            }
        }
        stage ('Make new release automatically') {
            agent { label 'docker' }
            when { branch 'release/new' }
            steps {
                jplMakeRelease(cfg, true)
            }
        }
        stage ('Release confirm') {
            when { branch 'release/v*' }
            steps {
                jplPromoteBuild(cfg)
            }
        }
        stage ('Release finish') {
            agent { label 'docker' }
            when { expression { cfg.BRANCH_NAME.startsWith('release/v') && cfg.promoteBuild.enabled } }
            steps {
                sh "docker rmi kairops/duing:test kairops/duing:19.04 || true"
                jplDockerPush (cfg, "kairops/duing", cfg.releaseTag, "duing", "https://registry.hub.docker.com", "cikairos-docker-credentials")
                jplDockerPush (cfg, "kairops/duing", "19.04", "duing", "https://registry.hub.docker.com", "cikairos-docker-credentials")
                jplDockerPush (cfg, "kairops/duing", "latest", "duing", "https://registry.hub.docker.com", "cikairos-docker-credentials")
                jplCloseRelease(cfg)
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
        skipDefaultCheckout()
        timeout(time: 1, unit: 'DAYS')
    }
}

#!/usr/bin/env groovy

podTemplate(
    name: 'whee', 
    cloud: 'kubernetes', 
    label: 'image-builder-maven-pg', 
    namespace: 'ci',
    inheritFrom: 'image-builder', 
    containers: [
        containerTemplate(name: 'jnlp', 
                          image: 'jenkinsci/jnlp-slave:2.62-alpine', 
                          envVars: [
                             containerEnvVar(key: 'JENKINS_NAME', value: '${computer.name}'),
                             containerEnvVar(key: 'JENKINS_SECRET', value: '${computer.jnlpmac}')
                          ]),
        containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat', args: ''),
        containerTemplate(name: 'postgres', 
                          image: 'postgres:9.5-alpine', 
                          ttyEnabled: true, 
                          command: 'cat', 
                          args: '',
                          envVars: [
                            containerEnvVar(key: 'POSTGRES_PASSWORD', value: 'postgres'),
                            containerEnvVar(key: 'POSTGRES_USER', value: 'postgres'),
                          ]),
        containerTemplate(name: 'docker-build', image: 'docker:1.12.3-dind', privileged: true)
    ])  {


    node('image-builder') {

        def commitAuthorEmail = sh(script: "git show -q --format='%aE' HEAD", returnStdout: true).trim()
        def gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
        def shortCommit = gitCommit.take(6)

        def pom = readMavenPom file: 'pom.xml' 

        def version = pom.version.replace("SNAPSHOT", "")
        version = VersionNumber(version + '${BUILD_DATE_FORMATTED, "yyyyMMdd"}-${BUILDS_TODAY}')

        version = VersionNumber(version + '${BUILD_DATE_FORMATTED, "yyyyMMdd"}-${BUILDS_TODAY}')

        stage("checkout") {
            checkout scm
        }

        state('test-db-setup') {
            container('postgres') {
                echo 'preload the pg test bits here'
            }
        }
        stage('project build and test') {
            try {
                sh "mvn -DreleaseVersion=${version} -DdevelopmentVersion=${pom.version} -DpushChanges=false -DlocalCheckout=false -DpreparationGoals=initialize -Dgoals=package release:prepare release:perform -B -P skip-tests"
                stage('unit tests') {
                    junit 'target/checkout/**/target/surefire-reports/*.xml'
                }
            }
            catch(e) {
                currentBuild.result = "FAILED"
                send_email_notification(commitAuthorEmail)
                throw e
            }
        }

        stage('docker image build') {
            echo 'things built here'
            stage('build container image') {
                container('docker-build') {
                    sh """
                    cd /home/jenkins/workspace/${JOB_NAME}
                    docker build --rm -t kube-registry.kube-system:5000/${JOB_NAME}:${BUILD_NUMBER} .
                    docker build --rm -t kube-registry.kube-system:5000/${JOB_NAME}:latest .
                    """
                    echo 'the build should be complete at this point'

                    stage('push container to registry')

                    sh """
                    cd /home/jenkins/workspace/${JOB_NAME}
                    docker push kube-registry.kube-system:5000/${JOB_NAME}:${BUILD_NUMBER} 
                    docker push kube-registry.kube-system:5000/${JOB_NAME}:latest
                    """
                }
            }
        }
    }
    }


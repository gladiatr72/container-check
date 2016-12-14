#!/usr/bin/env groovy

podTemplate(
    name: 'whee', 
    cloud: 'default', 
    label: 'image-builder', 
    namespace: 'ci',
    inheritFrom: 'jenkins-slave-pod', 
    containers: [
        containerTemplate(name: 'jnlp', 
                          image: 'jenkinsci/jnlp-slave:2.62-alpine', 
                          envVars: [
                             containerEnvVar(key: 'JENKINS_NAME', value: '${computer.name}'),
                             containerEnvVar(key: 'JENKINS_SECRET', value: '${computer.jnlpmac}')
                          ]),
        containerTemplate(name: 'python', image: 'python:2.7', ttyEnabled: true, command: 'cat', args: ''),
        containerTemplate(name: 'docker-build', image: 'docker:1.12.3-dind', privileged: true)
    ]) 


node('image-builder') {

    stage("checkout") {
        checkout scm
    }

    stage('docker image build') {
        echo 'things built here'
        stage('build container image') {
            container('docker-build') {
                sh """
                cd /home/jenkins/workspace/${JOB_NAME}
                docker build --rm -t kube-registry.kube-system:5000/efk-demo-flask:${BUILD_NUMBER} .
                docker build --rm -t kube-registry.kube-system:5000/efk-demo-flask:latest .
                """
                echo 'the build should be complete at this point'

                stage('push container to registry')

                sh """
                cd /home/jenkins/workspace/${JOB_NAME}
                docker push kube-registry.kube-system:5000/efk-demo-flask:${BUILD_NUMBER} 
                docker push kube-registry.kube-system:5000/efk-demo-flask:latest
                """
            }
        }
    }
}
//}


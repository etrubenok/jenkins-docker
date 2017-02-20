#!/usr/bin/groovy

def label = "buildpod.${env.JOB_NAME}.${env.BUILD_NUMBER}".replace('-', '_').replace('/', '_')
podTemplate(label: label, 
  serviceAccount: 'jenkins',
  podSelector: (key: 'jenkins', value: 'trie'),
  containers: [
    [name: 'aws-cli', image: 'garland/aws-cli-docker', command: 'cat', ttyEnabled: true, envVars: [
      [key: 'DOCKER_CONFIG', value:'/home/jenkins/.docker/'] ]],
    [name: 'client', image: '862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/vlocity/alpine-builder-clients:v0.0.1', command: 'cat', ttyEnabled: true, envVars: [
      [key: 'DOCKER_CONFIG', value: '/home/jenkins/.docker/'],
      [key: 'KUBERNETES_MASTER', value: 'kubernetes.default']]],
    [name: 'jnlp', image: 'iocanel/jenkins-jnlp-client:latest', command:'/usr/local/bin/start.sh', args: '${computer.jnlpmac} ${computer.name}', ttyEnabled: false,
      envVars: [[key: 'DOCKER_HOST', value: 'unix:/var/run/docker.sock']]]],
    volumes: [
      [$class: 'HostPathVolume', mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock']
    ], 
    annotations: [
        podAnnotation(key: 'scheduler.alpha.kubernetes.io/tolerations', value: '[{"key":"dedicated", "value":"jenkins"}]')
    ]) 
  {
    node(label) {
      git url: 'git@bitbucket.org:vloc/jenkins-fabric8-ci.git', credentialsId: 'git-global-key'

      stage("Building docker image for the Vlocity csutomized Jenkins Fabric8") {
        def dockerLogin = ''
        container(name: 'aws-cli') {
            dockerLogin = sh (
                script: "aws ecr get-login --region ap-southeast-2 --registry-ids 862135206070",
                returnStdout: true
            ).trim()
        }

        container(name: 'client') {
          sh "eval ${dockerLogin}"
          sh "docker build -t 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:v0.2.${env.BUILD_NUMBER} ."
          sh "docker tag 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:v0.2.${env.BUILD_NUMBER} 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:latest"
          sh "docker push 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:v0.2.${env.BUILD_NUMBER}"
          sh "docker push 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:latest"
          sh "docker rmi -f 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:v0.2.${env.BUILD_NUMBER}"
          sh "docker rmi -f 862135206070.dkr.ecr.ap-southeast-2.amazonaws.com/jenkins-docker:latest"
        }
      }
    }
}
#!groovy

node {
    stage('Clone sources') {
        checkout scm;
    }

    def REPO_NAME = "adservice"
    def DOCKER_CONTEXT = "${WORKSPACE}"

    env.BRANCH_NAME = env.BRANCH_NAME ? env.BRANCH_NAME : 'master';
    def imageOwner = "dmitrybuhtiyarov"
    def imageName = "${imageOwner}/${REPO_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}";
    def registryUrl = "https://registry.hub.docker.com";
    def registryCredentialsId = "dockerhub_id";

    def customImage;
    docker.withRegistry(registryUrl, registryCredentialsId) {
        stage('Build') {
            customImage = docker.build(imageName, DOCKER_CONTEXT);
        }
    }

    stage('Security scan') {
        sh "echo  '${imageName} ${DOCKER_CONTEXT}/Dockerfile' > anchore_images"
        anchore forceAnalyze: true, name: 'anchore_images'
    }

    docker.withRegistry(registryUrl, registryCredentialsId) {
        stage('Push') {
            customImage.push();
        }
    }

}

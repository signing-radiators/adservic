#!groovy

node {
    stage('Clone sources') {
        checkout scm;
    }

    def REPO_NAME = "adservice"
    def DOCKER_CONTEXT = "${WORKSPACE}"

    env.BRANCH_NAME = env.BRANCH_NAME ? env.BRANCH_NAME : 'master';
    def imageOwner = "dmitrybuhtiyarov"
    def deployTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    def securityScanTag = "securityScan-${env.BUILD_NUMBER}"
    def imageName = "${imageOwner}/${REPO_NAME}"
    def registryFqdn = "registry.hub.docker.com"
    def registryUrl = "https://${registryFqdn}"
    def registryCredentialsId = "dockerhub_id"

    stage('Sonarqube') {
        environment {
            scannerHome = tool 'sonarqube'
        }
        steps {
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
            timeout(time: 10, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
            }
        }
    }

    docker.withRegistry(registryUrl, registryCredentialsId) {
        def securityScanImage;
        stage('Build for security scan') {
            securityScanImage = docker.build("${imageName}:${securityScanTag}", DOCKER_CONTEXT);
        }
        stage('Push for security scan') {
            securityScanImage.push();
        }
    }

    stage('Security scan') {
        sh "echo  '${registryFqdn}/${imageName}:${securityScanTag} ${DOCKER_CONTEXT}/Dockerfile' > anchore_images"
        anchore bailOnFail: false, forceAnalyze: true, name: 'anchore_images'
    }

    docker.withRegistry(registryUrl, registryCredentialsId) {
        def image;
        stage('Build') {
            image = docker.build("${imageName}:${deployTag}", DOCKER_CONTEXT);
        }
        stage('Push') {
            image.push();
        }
    }

}

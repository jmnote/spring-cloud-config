#!groovy
node {
    def git
    def commitHash
    def buildImage

    stage('Checkout') {
        git = checkout scm
        commitHash = git.GIT_COMMIT
    }

    stage('Test') {
        try{
            bat 'gradlew check'
        } finally {
            junit 'build/test-results/**/*.xml'
        }
    }

    stage('Build') {
        bat 'gradlew build -x test'

    }

    stage('Build Docker Image') {
        buildImage = docker.build("hubtea/spring-cloud-config:${commitHash}")
    }

    stage('Archive') {
        parallel (
            "Archive Artifacts" : {
                archiveArtifacts artifacts: '**/build/libs/*.jar', fingerprint: true
            },

            "Docker Image Push" : {
                buildImage.push("${commitHash}")
                buildImage.push("latest")
            }
        )
    }

    stage('Kubernetes Deploy') {
        bat 'kubectl apply --namespace=development -f deployment.yaml'
    }
}

pipeline {
    agent any

    options {
        timestamps()
        timeout(time: 40, unit: 'MINUTES')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test and Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw -B clean verify'
            }
        }

        stage('Build Container Image') {
            steps {
                sh './mvnw -B -DskipTests jib:buildTar'
                sh 'rm -f target/jib-image.tar'
            }
        }
    }

    post {
        always {
            junit(
                testResults: '**/target/surefire-reports/*.xml',
                allowEmptyResults: true
            )
        }

        success {
            echo 'Tests, application build, and container image build succeeded!'
        }

        failure {
            echo 'Build failed. Check the Jenkins console output.'
        }
    }
}

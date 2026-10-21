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

        stage('Publish Container Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        umask 077
                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT

                        AUTH="$(printf '%s' "$DOCKERHUB_USERNAME:$DOCKERHUB_TOKEN" | base64 | tr -d '\\n')"

                        printf '{"auths":{"https://index.docker.io/v1/":{"auth":"%s"}}}\\n' "$AUTH" > "$DOCKER_CONFIG/config.json"

                        ./mvnw -B -DskipTests \
                          -Dimage="docker.io/${DOCKERHUB_USERNAME}/spring-petclinic-rest:${BUILD_NUMBER}" \
                          jib:build
                    '''
                }
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
            echo 'Tests passed and container image published!'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}

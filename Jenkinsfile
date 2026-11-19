pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {
        COMPOSE_PROJECT_NAME = 'fusionpact-devops-challenge'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh 'docker version --format "{{.Server.Version}}"'
                sh 'docker compose config -q'
            }
        }

        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d --remove-orphans'
            }
        }

        stage('Smoke test') {
            steps {
                sh '''
                    check() {
                        for i in $(seq 1 20); do
                            if curl -fsS -o /dev/null "$1"; then
                                echo "OK   $1"; return 0
                            fi
                            sleep 5
                        done
                        echo "FAIL $1"; return 1
                    }
                    check http://localhost:80/
                    check http://localhost:8000/docs
                    check http://localhost:9090/-/ready
                    check http://localhost:3000/api/health
                '''
            }
        }
    }

    post {
        success { echo 'Deployment succeeded.' }
        failure {
            echo 'Deployment failed, recent container logs follow.'
            sh 'docker compose ps || true'
            sh 'docker compose logs --tail=50 || true'
        }
        always {
            sh 'docker image prune -f || true'
        }
    }
}

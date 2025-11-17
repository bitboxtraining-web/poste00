
pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        REGISTRY = "vps-XXXXX.ovh.net:5000"
        SONAR_URL = "http://vps-XXXXX.ovh.net:9000"
    }

    stages {

        /* ----- 1. QUALITY GATE pour TOUT sauf main ----- */
        stage('SonarQube Analysis') {
            when { not { branch "main" } }
            steps {
                withSonarQubeEnv('sonar') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        /* ----- 2. BUILD pour TOUT ----- */
        stage('Build Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        /* ----- 3. TESTS : seulement develop + feature/* ----- */
        stage('Unit Tests') {
            when { 
                expression { 
                    env.BRANCH_NAME == "develop" ||
                    env.BRANCH_NAME.startsWith("feature/")
                }
            }
            steps {
                sh 'mvn test'
            }
        }

        /* ----- 4. BUILD DOCKER : uniquement develop, release/* et main ----- */
        stage('Docker Build') {
            when {
                anyOf {
                    branch "develop"
                    branch "main"
                    expression { env.BRANCH_NAME.startsWith("release/") }
                }
            }
            steps {
                sh """
                docker build -t ${REGISTRY}/backend:${BRANCH_NAME} .
                docker push ${REGISTRY}/backend:${BRANCH_NAME}
                """
            }
        }

        /* ----- 5. DEPLOY : seulement main ----- */
        stage('Deploy to Production') {
            when { branch "main" }
            steps {
                sh """
                ssh root@VM 'docker pull ${REGISTRY}/backend:main'
                ssh root@VM 'docker compose -f /srv/app/docker-compose.yml up -d'
                """
            }
        }
    }

    /*
     * OPTIONS : notifications / clean workspace / discard old builds
     */
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }
}


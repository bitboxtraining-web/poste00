
pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        REGISTRY = "vps-36f602ea.vps.ovh.net:5000"
        SONAR_URL = "http://vps-XXXXX.ovh.net:9000"
    }

    stages {

        /* ----- 1. QUALITY GATE pour TOUT sauf main ----- */
         stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    dir('backend-demo') {
                        sh '''
                            mvn clean verify sonar:sonar                               -Dsonar.projectKey=demo-backend                               -Dsonar.host.url=$SONAR_HOST_URL                               -DskipTests
                        '''
                    }
                }
            }
        }

        /* ----- 2. BUILD pour TOUT ----- */
        stage('Build Maven') {
            
            steps {

                dir('backend-demo') {
                        sh '''
                            mvn clean package -DskipTests   
                    '''
                    }
                
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
        dir('backend-demo') {
            sh """
            docker build -t ${REGISTRY}/backend:${BRANCH_NAME} .
            docker push ${REGISTRY}/backend:${BRANCH_NAME}
            """
        }
    }
        }

      /* ----- 5. DEPLOY : seulement main ----- */
stage('Deploy to Production') {
    when { branch "main" }
    steps {
        sh """
        ssh -o StrictHostKeyChecking=no debian@217.182.207.167 '
            sudo docker pull ${REGISTRY}/backend:main &&
            sudo docker stop backend || true &&
            sudo docker rm backend || true &&
            sudo docker run -d --name backend -p 8080:8080 ${REGISTRY}/backend:main
        '
        """
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


pipeline {
    agent any

    environment {
        TARGET_DIR = 'C:\\Users\\ashflaare\\Desktop\\study\\4_c\\devops\\project_serv'
        REPO_URL = 'https://github.com/AshFlaare/task_sharing_management_system_new.git'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Clone or Update Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    bat """
                        if not exist "${TARGET_DIR}\\.git" (
                            echo Cloning fresh repo...
                            rmdir /S /Q "${TARGET_DIR}" 2>nul || echo No old folder
                            git clone -b fix https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git "${TARGET_DIR}"
                        ) else (
                            echo Updating existing repo...
                            cd "${TARGET_DIR}"
                            git reset --hard
                            git clean -fd
                            git pull https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git fix
                        )
                    """
                }
            }
        }

        stage('Run Backend Tests') {
            steps {
                bat """
                    cd "${TARGET_DIR}"
                    docker compose run --rm backend python manage.py test
                """
            }
        }

        stage('Build Containers') {
            steps {
                bat """
                    cd "${TARGET_DIR}"
                    docker compose build
                """
            }
        }

        stage('Restart Application') {
            steps {
                bat """
                    cd "${TARGET_DIR}"
                    docker compose down
                    docker compose up -d
                """
            }
        }

        stage('Push Docker Images to Local Registry') {
            when {
                expression { return fileExists("${TARGET_DIR}\\\\docker-compose.yml") }
            }
            steps {
                bat """
                    echo Pushing Docker images to local registry...
                    docker tag backend localhost:5000/backend:latest
                    docker tag frontend localhost:5000/frontend:latest
                    docker tag nginx localhost:5000/nginx:latest

                    docker push localhost:5000/backend:latest
                    docker push localhost:5000/frontend:latest
                    docker push localhost:5000/nginx:latest
                """
            }
        }
    }

    post {
        success {
            echo "Build & Tests passed successfully!"
            echo "Containers rebuilt and restarted."
            echo "Backend: http://localhost:8000/"
            echo "Frontend (via Nginx): http://localhost/"
        }
        failure {
            echo "Tests failed or build error occurred."
        }
    }
}

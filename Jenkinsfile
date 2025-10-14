pipeline {
    agent any

    environment {
        TARGET_DIR = 'C:\\Users\\ashflaare\\Desktop\\study\\4_c\\devops\\project_serv'
        REPO_URL = 'https://github.com/AshFlaare/task_sharing_management_system_new.git'
        BUILD_VERSION = "${BUILD_NUMBER}"
        REGISTRY = "localhost:5000"
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

        stage('Merge fix -> main') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN'),
                    string(credentialsId: 'github-email', variable: 'GIT_EMAIL')
                ]) {
                    bat """
                        cd "${TARGET_DIR}"
                        git config user.name "%GIT_USER%"
                        git config user.email "%GIT_EMAIL%"

                        echo Checking out main branch...
                        git checkout main
                        git pull https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git main

                        echo Merging fix -> main...
                        git merge fix -m "Auto-merge from Jenkins after successful tests"

                        echo Pushing main...
                        git push https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git main

                        echo Syncing fix with main...
                        git checkout fix
                        git merge main -m "Sync fix with main"
                        git push https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git fix
                    """
                }
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

        stage('Tag & Push Docker Images to Local Registry') {
            steps {
                bat """
                    echo Tagging and pushing images to registry...

                    docker tag backend ${REGISTRY}/backend:build-${BUILD_VERSION}
                    docker tag nginx ${REGISTRY}/nginx:build-${BUILD_VERSION}

                    docker push ${REGISTRY}/backend:build-${BUILD_VERSION}
                    docker push ${REGISTRY}/nginx:build-${BUILD_VERSION}

                    echo Also update latest tags...
                    docker tag backend ${REGISTRY}/backend:latest
                    docker tag nginx ${REGISTRY}/nginx:latest
                    docker push ${REGISTRY}/backend:latest
                    docker push ${REGISTRY}/nginx:latest
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
    }

    post {
        success {
            echo "Build & Tests passed!"
            echo "Code merged fix → main."
            echo "Containers tagged and pushed as build-${BUILD_NUMBER}"
            echo "Backend restarted at http://localhost:8000/"
            echo "Frontend via Nginx at http://localhost/"
        }
        failure {
            echo "Tests failed, merge and deployment skipped."
        }
    }
}

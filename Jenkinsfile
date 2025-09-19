pipeline {
    agent any

    environment {
        CMD = 'C:\\Windows\\System32\\cmd.exe'
        PM2_CMD = 'C:\\Users\\ashflaare\\AppData\\Roaming\\npm\\pm2.cmd'
        PYTHON_EXE = 'C:\\Program Files\\Python313\\python.exe'
        TARGET_DIR = 'C:\\Users\\ashflaare\\Desktop\\study\\4_c\\devops\\project_serv'
        REPO_URL = 'https://github.com/AshFlaare/task_sharing_management_system_new.git'
    }

    triggers { 
        githubPush() 
    }

    stages {
        stage('Checkout code') {
            when { branch 'fix' }
            steps {
                git branch: 'fix',
                    url: "${REPO_URL}",
                    credentialsId: 'github-creds'
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                    cd "${TARGET_DIR}"
                    "${PYTHON_EXE}" application\\integrationtests.py
                """
            }
        }

        stage('Merge fix -> main') {
            when {
                branch 'fix'
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

                        git checkout main
                        git pull https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git main

                        git merge fix

                        git push https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git main

                        git checkout fix
                        git merge main
                        git push https://%GIT_USER%:%GIT_TOKEN%@github.com/AshFlaare/task_sharing_management_system_new.git fix
                    """
                }
            }
        }

        stage('Restart Servers') {
            steps {
                bat """
                    call "${PM2_CMD}" delete django || echo No Django process
                    call "${PM2_CMD}" start "${PYTHON_EXE}" --name django -- manage.py runserver 127.0.0.1:8000

                    call "${PM2_CMD}" delete vue || echo No Vue process
                    call "${PM2_CMD}" start "${CMD}" --name vue -- /c "cd ${TARGET_DIR}\\client && npm run dev"
                """
            }
        }
    }

    post {
        success {
            echo "✅ Build & Tests passed!"
            echo "✅ Code merged fix → main."
            echo "✅ Backend and Frontend restarted via PM2."
            echo "Backend: http://127.0.0.1:8000/"
            echo "Frontend: http://127.0.0.1:5173/"
        }
        failure {
            echo "❌ Tests failed, merge skipped, servers not updated."
        }
    }
}

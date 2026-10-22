pipeline {
    agent any
    environment {
        PYTHON = 'C:\\Users\\User\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'
        VENV = '.venv-ci'
        REPO = 'https://github.com/Glebwen/DevOPSLab1.git'
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Setup') {
            steps {
                bat """
                    if not exist %VENV% ("${PYTHON}" -m venv %VENV%)
                    "%VENV%\\Scripts\\python.exe" -m pip install -r requirements.txt
                """
            }
        }
        stage('Test') {
            steps {
                bat """
                    if not exist reports mkdir reports
                    "%VENV%\\Scripts\\python.exe" -m pytest --junitxml=reports/junit.xml
                """
            }
            post { always { junit 'reports/junit.xml' } }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                bat """
                    if not exist C:\\deploy mkdir C:\\deploy
                    if exist C:\\deploy\\DevOPSLab1 (
                        cd /d C:\\deploy\\DevOPSLab1 & git pull
                    ) else (
                        git clone %REPO% C:\\deploy\\DevOPSLab1
                    )
                    cd /d C:\\deploy\\DevOPSLab1
                    if not exist .venv ("${PYTHON}" -m venv .venv)
                    .venv\\Scripts\\python.exe -m pip install -r requirements.txt
                    .venv\\Scripts\\python.exe manage.py migrate
                """
            }
        }
    }
}
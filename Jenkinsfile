pipeline {
    agent any
    
    environment {
        PYTHON = 'C:\\Users\\User\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'
        VENV = '.venv-ci'
        REPO = 'https://github.com/Glebwen/DevOPSLab1.git'
        BUILD_ID = 'dontKillMe' 
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

                    :: Освобождаем порты
                    FOR /F "tokens=5" %%T IN ('netstat -a -n -o ^| findstr :8000') DO taskkill /F /PID %%T 2>nul
                    FOR /F "tokens=5" %%T IN ('netstat -a -n -o ^| findstr :5173') DO taskkill /F /PID %%T 2>nul

                    :: Ставим зависимости
                    if not exist .venv ("${PYTHON}" -m venv .venv)
                    .venv\\Scripts\\python.exe -m pip install -r requirements.txt
                    .venv\\Scripts\\python.exe manage.py migrate
                    
                    cd client
                    call npm install
                    cd ..

                    :: СОЗДАЕМ И ЗАПУСКАЕМ БЭКЕНД В НОВОМ ОКНЕ
                    start "Django_Deploy" cmd /c ".venv\\Scripts\\python.exe manage.py runserver 0.0.0.0:8000 > backend.log 2>&1"
                    
                    :: СОЗДАЕМ И ЗАПУСКАЕМ ФРОНТЕНД В НОВОМ ОКНЕ
                    start "Vite_Deploy" cmd /c "cd client && npm run dev > frontend.log 2>&1"
                    
                    :: Даем серверам 5 секунд, чтобы подняться
                    ping 127.0.0.1 -n 6 > nul
                """
            }
        }
    }
}
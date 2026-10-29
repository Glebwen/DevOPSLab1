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

                    :: 1. Останавливаем старые процессы, чтобы освободить порты
                    FOR /F "tokens=5" %%T IN ('netstat -a -n -o ^| findstr :8000') DO taskkill /F /PID %%T 2>nul
                    FOR /F "tokens=5" %%T IN ('netstat -a -n -o ^| findstr :5173') DO taskkill /F /PID %%T 2>nul

                    :: 2. Устанавливаем зависимости
                    if not exist .venv ("${PYTHON}" -m venv .venv)
                    .venv\\Scripts\\python.exe -m pip install -r requirements.txt
                    .venv\\Scripts\\python.exe manage.py migrate
                    
                    cd client
                    call npm install
                    cd ..

                    :: 3. Создаем bat-файл, который запустит серверы
                    echo @echo off > C:\\deploy\\start_all.bat
                    echo cd /d C:\\deploy\\DevOPSLab1 >> C:\\deploy\\start_all.bat
                    echo start "Django Backend" cmd /c ".venv\\Scripts\\python.exe manage.py runserver" >> C:\\deploy\\start_all.bat
                    echo cd client >> C:\\deploy\\start_all.bat
                    echo start "Vite Frontend" cmd /c "npm run dev" >> C:\\deploy\\start_all.bat
                    
                    :: 4. Создаем VBS-скрипт, который запустит этот bat-файл СКРЫТО и отвяжет от Дженкинса
                    echo Set WshShell = CreateObject("WScript.Shell") > C:\\deploy\\run.vbs
                    echo WshShell.Run "C:\\deploy\\start_all.bat", 0, False >> C:\\deploy\\run.vbs
                    
                    :: 5. Запускаем VBS. Дженкинс больше не сможет убить эти процессы!
                    wscript C:\\deploy\\run.vbs
                    
                    :: 6. Ждем 5 секунд, чтобы серверы успели подняться
                    ping 127.0.0.1 -n 6 > nul
                """
            }
        }
    }
}
pipeline {
    parameters {
        string(name: 'sleep_time', defaultValue: "4", description: 'Time to sleep after build stage')
        string(name: 'keep_alive_minutes', defaultValue: '0', description: 'How long to keep Flask app alive (in minutes)')
    }

    agent {
        label 'git-agent'
    }

    stages {
        stage('pre-Build') {
            steps {
                sh '''
                    set -e
                    echo "== Disabling APT timestamp validity check =="
                    sudo sh -c 'echo Acquire::Check-Valid-Until \\"false\\"; > /etc/apt/apt.conf.d/99no-check-valid-until'
        
                    echo "== Updating system packages =="
                    sudo apt-get update
        
                    echo "== Installing required packages =="
                    sudo apt-get install -y python3 python3-venv python3-flask python3-dev git curl binutils python3-pip pipx pylint
        
                    export PATH="$HOME/.local/bin:$PATH"
                    echo "== PATH set to: $PATH"
                '''
            }
        }



        stage('install-pyinstaller') {
            steps {
                sh '''
                    set -e
                    export PATH="$HOME/.local/bin:$PATH"
        
                    echo "== Forcing reinstallation of pyinstaller =="
                    pipx install --force pyinstaller
                '''
            }
        }

        stage('lint') {
            steps {
                sh '''
                    echo "== Running pylint on app.py =="
                    pylint --disable=missing-docstring,invalid-name app.py
                '''
            }
        }

        stage('build') {
            steps {
                sh '''
                    export PATH="$HOME/.local/bin:$PATH"
                    echo "== Building with pyinstaller =="
                    pyinstaller app.py -y || { echo "PyInstaller build failed"; exit 1; }
                '''
            }
        }
        
        stage('test') {
            steps {
                sh """
                    set -e
                    echo "== Starting Flask app in background =="
                    nohup python3 app.py > flask.log 2>&1 &
                    APP_PID=\$!
        
                    echo "== Waiting ${params.sleep_time} seconds for app to start =="
                    sleep ${params.sleep_time}
        
                    echo "== Testing endpoints =="
                    if curl -s localhost:8000 > /dev/null; then
                        echo 'Basic route test: success'
                    else
                        echo 'Basic route test: fail'
                        kill \$APP_PID
                        exit 1
                    fi
        
                    if curl -s localhost:8000/jenkins > /dev/null; then
                        echo 'Custom route test: success'
                    else
                        echo 'Custom route test: fail'
                        kill \$APP_PID
                        exit 1
                    fi
        
                    echo "== Holding Flask app for ${params.keep_alive_minutes} minute(s) =="
                    sleep \$(( ${params.keep_alive_minutes} * 60 ))
        
                    echo "== Done. Killing Flask app =="
                    kill \$APP_PID || true
                """
            }
        }

    }

    post {
        unsuccessful {
            echo "Build failed — cleaning workspace"
            cleanWs cleanWhenSuccess: false
        }
        success {
            echo "Build succeeded — archiving artifacts"
            archiveArtifacts artifacts: 'dist/**', followSymlinks: false
        }
    }
}

pipeline {
    parameters {
        string(name: 'sleep_time', defaultValue: "4", description: 'Time to sleep after build stage')
    }

    agent {
        label 'git-agent'
    }

    stages {
        stage('pre-Build') {
            steps {
                sh '''
                    set -e
        
                    echo "== Fixing system time =="
                    sudo apt-get install -y ntpdate
                    sudo ntpdate time.google.com
        
                    echo "== Updating system packages =="
                    sudo apt-get update
        
                    echo "== Installing required packages =="
                    sudo apt-get install -y python3 python3-venv python3-flask python3-dev git curl binutils python3-pip pipx pylint
        
                    echo "== PATH before modifying =="
                    echo $PATH
        
                    export PATH="$HOME/.local/bin:$PATH"
        
                    echo "== PATH after adding ~/.local/bin =="
                    echo $PATH
                '''
            }
        }


        stage('install-pyinstaller') {
            steps {
                sh '''
                    set -e
                    export PATH="$HOME/.local/bin:$PATH"

                    echo "== Checking if pyinstaller is already installed =="
                    if ! command -v pyinstaller > /dev/null; then
                        echo "Installing pyinstaller with pipx..."
                        pipx install pyinstaller
                    else
                        echo "PyInstaller is already installed at: $(which pyinstaller)"
                    fi

                    echo "== pyinstaller binary =="
                    ls -l $(which pyinstaller) || echo "pyinstaller not found"
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
                    echo "== Starting app for test =="
                    python3 app.py &
                    APP_PID=\$!

                    echo "== Sleeping for ${params.sleep_time} seconds =="
                    sleep ${params.sleep_time}

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

                    echo "== Killing app process after tests =="
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

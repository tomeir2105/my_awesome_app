pipeline {
    parameters {
        string(name: 'sleep_time', defaultValue: "4", description: 'Time to sleep after build stage')
        // choice(name: 'system_name', choices: ['worker1', 'worker2'], description: 'Agent name')
    }

    // agent { label "${params.system_name}" }
    agent {
        label 'git-agent'
    }

    stages {
        stage('pre-Build') {
            steps {
                sh '''
                    set -e
                    sudo apt-get update
                    sudo apt-get install -y python3 python3-venv python3-flask python3-dev git curl binutils python3-pip pipx pylint

                    echo "Using pipx from: $(which pipx)"
                    pipx list | grep -q pyinstaller || pipx install pyinstaller
                '''
            }
        }

        stage('lint') {
            steps {
                sh '''
                    echo "Running pylint on app.py"
                    pylint --disable=missing-docstring,invalid-name app.py
                '''
            }
        }

        stage('build') {
            steps {
                sh '''
                    echo "Building with pyinstaller"
                    pyinstaller app.py -y
                '''
            }
        }

        stage('test') {
            steps {
                sh """
                    echo "Starting app for test..."
                    python3 app.py &
                    APP_PID=\$!

                    echo "Sleeping for ${params.sleep_time} seconds"
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

                    echo "Killing app process after tests"
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

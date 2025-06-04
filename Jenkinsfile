pipeline{
	parameters {
		string(name: 'sleep_time', defaultValue: "2", description: 'time to sleep after build stage')
		choice(name: 'system_name', choices: ['worker1', 'worker2'], description: 'Agent name')
	}
	agent {label "${params.system_name}"}
	/*
	agent {
		dockerContainer {
			image 'openjdk:latest'
		}
	}
	*/
	stages{
		stage('pre-Build'){
			steps{
			sh '''
			sudo apt-get update 
			sudo apt-get install -y python3 python3-flask git pylint pipx
			pipx install pyinstaller
			'''		
			}		
		}
		stage('lint'){
			steps{
			sh '''
				pylint --disable=missing-docstring,invalid-name app.py
			'''
			}		
		}

		stage('build'){
			steps{
				sh """                 
					python3 app.py &
		 			chmod 777 /home/jenkins/.local/bin/pyinstaller
		    		/home/jenkins/.local/bin/pyinstaller app.py -y
					sleep ${params.sleep_time}
				"""
			}
		}
	
		stage('test'){
			steps{
				sh '''
				set -e
	            if curl localhost:8000 &> /dev/null;then
                        echo 'post test: success'
                    else
                        echo 'post test: fail'
                        exit 1
                    fi
                    if curl localhost:8000/jenkins &> /dev/null;then
                        echo 'post test with variable: success'
                    else
                        echo 'post test with variable: fail'
                        exit 1
                    fi
				'''
	
			}		
		}
	}
	post {
		unsuccessful{
			cleanWs cleanWhenSuccess: false
		}
		success{
				archiveArtifacts artifacts: 'dist/', followSymlinks: false
		}
	}
}




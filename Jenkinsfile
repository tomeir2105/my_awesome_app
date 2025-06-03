pipeline{
	agent any
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
				sh '''                   
					python3 app.py &
		 			chmod 777 /home/jenkins/.local/bin/pyinstaller
		    		/home/jenkins/.local/bin/pyinstaller  app.py -y
				'''
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
			cleanWs cleanWhenSiccessful: false
		}
		success{
				archiveArtifacts artifacts: 'dist/', followSymlinks: false
		}
	}
}




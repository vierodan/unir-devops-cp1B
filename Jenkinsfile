pipeline {
    agent any

    stages {
	    stage('Stash files') {
            steps {
                stash includes: '**/*', name: 'sourceCode'
            }
        }
	    stage('Mock Build') {
            steps {
                echo 'Eyyy this is Python is not necessary to compile anything'
                bat '''
                    echo %WORKSPACE%
                    dir
                '''
            }
        }
		stage('Unit Tests') {
		    agent {
                label 'agt2'
            }
			steps {
				unstash 'sourceCode'
				bat '''
					hostname
					whoami
					echo %WORKSPACE%
				'''
				catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
					bat '''
						set PYTHONPATH=%WORKSPACE%
						pytest --junitxml=result-unit.xml test\\unit
					'''
				}
			}
            post{
                always{
                    stash includes: 'result-unit.xml', name: 'results_unit'
                }
            }
		}
		stage('Run Flask and Wiremock') {
			agent {
                label 'agt3'
            }
			steps {
				unstash 'sourceCode'
				bat '''
					hostname
					whoami
					echo %WORKSPACE%
				'''
				catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
					bat '''
						set PYTHONPATH=%WORKSPACE%
						set FLASK_APP=app\\api.py
						start flask run
						start java -jar C:\\avr\\wiremock\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir test\\wiremock
                    '''
						sleep(time: 10, unit: 'SECONDS')
                    bat '''
						set PYTHONPATH=%WORKSPACE%
						pytest --junitxml=result-rest.xml test\\rest
					'''
				}
			}
            post{
                always{
                    stash includes: 'result-rest.xml', name: 'results_rest'
                }
            }
		}
        stage('Results') {
            steps {
                unstash 'results_unit'
                unstash 'results_rest'
                sleep(time: 5, unit: 'SECONDS')
                junit 'result*.xml'
                echo 'Finish'
            }
        }
    }
}
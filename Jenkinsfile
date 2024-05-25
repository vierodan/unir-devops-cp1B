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
 
    }
}
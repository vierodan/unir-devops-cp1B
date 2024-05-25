pipeline {
    agent any
    stages {
        stage('Mock Build') {
            steps {
                echo 'Eyyy this is Python is not necessary to compile anything'
                echo env.WORKSPACE
                bat 'dir'
            }
        }
        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-unit.xml test\\unit
                            '''
                        }
                    }
                }
                stage('Run Flask and Wiremock') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                set FLASK_APP=app\\api.py
                                start flask run
                                start java -jar C:\\avr\\wiremock\\wiremock-standalone-3.5.4.jar --port 9090 --root-dir test\\wiremock
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Results') {
            steps {
                junit 'result*.xml'
                echo 'Finish'
            }
        }
    }
}
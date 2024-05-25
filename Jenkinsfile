pipeline {
    agent any

    stages {
        stage('Tests') {
            parallel {
                stage('Static Analysis'){
                    steps{
                        bat '''
                            hostname
                            whoami
                            echo %WORKSPACE%

                            flake8 --exit-zero --format=pylint --max-line-length=120 app > flake8.out
                        '''

                        recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates:[[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
                    }
                }
                stage('Unit Tests') {
                    steps {
                        bat '''
                            hostname
                            whoami
                            echo %WORKSPACE%
                        '''
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                coverage run --source=app --omit=app\\__init__.py,app\\api.py -m pytest --junitxml=result-unit.xml test\\unit
                                coverage xml
                            '''
                        }
                    }
                }
                stage('Api Tests') {
                    steps {
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
                                sleep(time: 15, unit: 'SECONDS')
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Coverage'){
            steps{
                bat '''
                    hostname
                    whoami
                    echo %WORKSPACE%
                '''
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,90,80', lineCoverageTargets: '100,95,85', onlyStable: false
                }
            }
        }
        stage('Results') {
            steps {
                sleep(time: 5, unit: 'SECONDS')
                junit 'result*.xml'
                echo 'Finish'
            }
        }
    }
}
pipeline {
    agent {
        label 'agent3'
    }

    stages {
        stage('Tests') {
            parallel {
                stage('Static Code Analysis'){
                    steps{
                        bat '''
                            hostname
                            whoami
                            echo %WORKSPACE%
                        '''
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                flake8 --exit-zero --format=pylint --max-line-length=120 app > flake8.out
                            '''

                            recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], 
                                qualityGates:[[threshold: 8, type: 'TOTAL', unstable: true], 
                                              [threshold: 10, type: 'TOTAL', unstable: false]]
                        }
                    }
                }
                stage('Unit Tests and Coverage') {
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

                            cobertura coberturaReportFile: 'coverage.xml', 
                                conditionalCoverageTargets: '100,90,80', 
                                lineCoverageTargets: '100,95,85', 
                                onlyStable: false  
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
                                sleep(time: 12, unit: 'SECONDS')
                            bat '''
                                set PYTHONPATH=%WORKSPACE%
                                pytest --junitxml=result-rest.xml test\\rest
                            '''
                        }
                    }
                }
                stage('Scurity Analysis'){
                    steps{
                        bat '''
                            hostname
                            whoami
                            echo %WORKSPACE%
                        '''
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            bat '''
                                bandit --exit-zero -r . -f custom -o bandit.out --severity-level medium --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                            '''

                            recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], 
                                                    qualityGates:[[threshold: 3, type: 'TOTAL', unstable: true], 
                                                                [threshold: 4, type: 'TOTAL', unstable: false]]

                        }
                    }
                }
            }
        }
        stage('Performance Analysis'){
            steps{
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
                    '''
                        sleep(time: 12, unit: 'SECONDS')
                    bat '''
                        jmeter -n -t test\\jmeter\\add-plan.jmx -f -l add.jtl
                        jmeter -n -t test\\jmeter\\substract-plan.jmx -f -l substract.jtl
                    '''

                    perfReport sourceDataFiles: 'add.jtl'
                    perfReport sourceDataFiles: 'substract.jtl'
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
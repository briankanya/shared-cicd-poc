#!/usr/bin/env groovy

pipeline {
    agent any

    stages {

        stage ('Install Requirements') {
            steps {
                sh """
                    cat /etc/*release
                    #virtualenv --python=python2.7 venv
                    virtualenv venv
                    #. venv/bin/activate
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    pip install --upgrade pip
                    pip install -r requirements.txt -r dev-requirements.txt
                    make clean
                """
            }
        }

        stage ('Check Style') {
            steps {
                sh """
                    #. venv/bin/activate
                    [ -d report ] || mkdir report
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    make check || true
                """
                sh """
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    make flake8 | tee report/flake8.log || true
                """
                sh """
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    make pylint | tee report/pylint.log || true
                """
                step([$class: 'WarningsPublisher',
                  parserConfigurations: [[
                    parserName: 'Pep8',
                    pattern: 'report/flake8.log'
                  ],
                  [
                    parserName: 'pylint',
                    pattern: 'report/pylint.log'
                  ]],
                  unstableTotalAll: '0',
                  usePreviousBuildAsReference: true
                ])
            }
        }

        stage ('Unit Tests') {
            steps {
                sh """
                    #. venv/bin/activate
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    make unittest || true
                """
            }

            post {
                always {
                    junit keepLongStdio: true, testResults: 'report/nosetests.xml'
                    publishHTML target: [
                        reportDir: 'report/coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report - Unit Test'
                    ]
                }
            }
        }

        stage ('System Tests') {
            steps {
                sh """
                    #. venv/bin/activate
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    // Write file containing test node connection information if needed.
                    // writeFile file: "test/fixtures/nodes.yaml", text: "---\n- node: <some-ip>\n"
                    make systest || true
                """
            }

            post {
                always {
                    junit keepLongStdio: true, testResults: 'report/nosetests.xml'
                    publishHTML target: [
                        reportDir: 'report/coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report - System Test'
                    ]
                }
            }
        }

        stage ('Docs') {
            steps {
                sh """
                    #. venv/bin/activate
                    export PATH=${VIRTUAL_ENV}/bin:${PATH}
                    PYTHONPATH=. pdoc --html --html-dir docs --overwrite env.projectName
                """
            }

            post {
                always {
                    publishHTML target: [
                        reportDir: 'docs/*',
                        reportFiles: 'index.html',
                        reportName: 'Module Documentation'
                    ]
                }
            }
        }

        stage ('Cleanup') {
            steps {
                sh 'rm -rf venv'
            }
        }
    }
}
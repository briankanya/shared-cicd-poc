#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        projectName = 'ProjectTemplate'
    }

    stages {

        stage ('Install Requirements') {
            steps {
                sh """
                    apt update -y
                    dpkg --configure -a
                    apt install -y python3-pip
                    pip install --upgrade pip
                    pip install -r requirements.txt -r dev-requirements.txt
                    make clean
                """
            }
        }

        stage ('Check Style') {
            steps {
                sh """
                    make check || true
                """
                sh """
                    make flake8 | tee report/flake8.log || true
                """
                sh """
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
                    pdoc --html --html-dir docs --overwrite env.projectName
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
                sh 'ls -R .'
            }
        }
    }
}
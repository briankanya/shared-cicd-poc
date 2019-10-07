#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        projectName = 'python-jenkinsfile-testing'
        emailTo = 'briankanya@gmail.com'
        emailFrom = 'eosplus-dev+jenkins@arista.com'
    }

    stages {

        stage ('Install Requirements') {
            steps {
                sh """
                    apt update -y
                    dpkg --configure -a
                    apt install -y python3-pip
                    pip3 install -r requirements.txt -r dev-requirements.txt
                    mkdir -p report
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
    }
}
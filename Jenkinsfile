#!/usr/bin/env groovy

pipeline {
    agent any

    environment {
        projectName = 'python-jenkinsfile-testing'
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
        }

        stage ('System Tests') {
            steps {
                sh """
                    make systest || true
                """
            }
        }
    }
}
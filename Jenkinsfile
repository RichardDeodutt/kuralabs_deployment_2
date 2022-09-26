pipeline {
  agent any
    stages {
      stage ('Build Tools') {
        steps {
          sh '''#!/bin/bash
          source testenv/bin/activate
          node --max-old-space-size=400 /usr/bin/npm install --save-dev cypress@8.7.0
          /usr/bin/npx cypress verify
          '''
        }
      }
      stage ('Build App') {
        steps {
          sh '''#!/bin/bash
          python3 -m venv testenv
          source testenv/bin/activate
          pip install pip --upgrade
          pip install -r requirements.txt
          export FLASK_APP=application
          flask run &
          '''
        }
      }
      stage ('Pytest') {
        steps {
          sh '''#!/bin/bash
            source testenv/bin/activate
            py.test --verbose --junit-xml test-reports/pytest-results.xml
            '''
        }
        post{
          always {
            junit 'test-reports/pytest-results.xml'
          }
        }
      }
      stage ('Pylint') {
        steps {
          sh '''#!/bin/bash
            source testenv/bin/activate
            pylint --output-format=text,pylint_junit.JUnitReporter:test-reports/pylint-results.xml application.py
            '''
        }
        post{
          always {
            junit 'test-reports/pylint-results.xml'
          }
        }
      }
      stage ('Deploy') {
        steps {
          sh '/var/lib/jenkins/.local/bin/eb deploy url-shortener-dev'
        }
      }
      stage ('Cypress E2E') {
        steps {
          sh '''#!/bin/bash
            source testenv/bin/activate
            /usr/bin/npx cypress run --spec cypress/integration/test.spec.js
            '''
        }
        post{
          always {
            junit 'test-reports/cypress-results.xml'
          }
        }
      }
    }
  }

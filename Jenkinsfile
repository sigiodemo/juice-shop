def BRIDGE = 'https://sig-repo.synopsys.com/artifactory/bds-integrations-release/com/synopsys/integration/synopsys-action/0.1.72/ci-package-0.1.72-linux64.zip'

pipeline {
  agent any

  environment {
    APPLICATION = 'MJ-test-projects'
    PROJECT = 'juice-shop'
    VERSION = '1.0'
    BRANCH = 'main'
    WORKSPACE_TMP = '/tmp'

    SEEKER_TOKEN = credentials('SEEKER_TOKEN')
    SERVER_START = "npm test"
    SEEKER_STUFF = "java -javaagent:seeker/seeker-agent.jar -jar target/jhipster-sample-application-0.0.1-SNAPSHOT.jar"
    SERVER_STRING = "Application 'jhipsterSampleApplication' is running!"
    SERVER_WORKINGDIR = ""
    SEEKER_RUN_TIME = 180
    SEEKER_PROJECT_KEY = 'jshop'
  }

  stages{
    stage('Set Up Environment') {
      steps {
        sh '''
          curl -s -L https://raw.githubusercontent.com/jones6951/io-scripts/main/getProjectID.sh > /tmp/getProjectID.sh
          curl -s -L https://raw.githubusercontent.com/jones6951/io-scripts/main/serverStart.sh > /tmp/serverStart.sh
          curl -s -L https://raw.githubusercontent.com/jones6951/io-scripts/main/isNumeric.sh > /tmp/isNumeric.sh
          curl -s -L https://raw.githubusercontent.com/synopsys-sig/io-artifacts/main/prescription.sh > /tmp/prescription.sh

          chmod +x /tmp/getProjectID.sh
          chmod +x /tmp/serverStart.sh
          chmod +x /tmp/isNumeric.sh
          chmod +x /tmp/prescription.sh
        '''
      }
    }

    stage('NPM Install') {
      steps {
        sh 'npm install'
      }
    }

    stage('Polaris') {
      steps {
        withCredentials([string(credentialsId: 'POLARIS_TOKEN', variable: 'BRIDGE_POLARIS_ACCESSTOKEN')]) {
          script {
            status = sh returnStatus: true, script: """
              export BRIDGE_POLARIS_SERVERURL=$POLARIS_SERVER_URL
              export BRIDGE_POLARIS_APPLICATION_NAME=$APPLICATION
              export BRIDGE_POLARIS_PROJECT_NAME=$PROJECT

              curl -fLsS -o $WORKSPACE_TMP/bridge.zip $BRIDGE
              unzip -qo -d $WORKSPACE_TMP/bridge $WORKSPACE_TMP/bridge.zip
              $WORKSPACE_TMP/bridge/bridge --stage polaris polaris.assessment.types='["SAST","SCA"]'
            """
            if (status == 8) { unstable 'policy violation' }
            else if (status != 0) { error 'bridge failure' }
          }
        }
      }
    }

    stage ('IAST - Seeker') {
      steps {
        withCredentials([string(credentialsId: 'SEEKER_TOKEN', variable: 'SEEKER_TOKEN')]) {
        '''#!/bin/bash
          if [ ! -z ${SERVER_WORKINGDIR} ]; then cd ${SERVER_WORKINGDIR}; fi

          sh -c "$( curl -k -X GET -fsSL --header 'Accept: application/x-sh' \"${SEEKER_SERVER_URL}/rest/api/latest/installers/agents/scripts/JAVA?osFamily=LINUX&downloadWith=curl&projectKey=${SEEKER_PROJECT_KEY}&webServer=TOMCAT&flavor=DEFAULT&agentName=&accessToken=\")"

          export SEEKER_PROJECT_VERSION=${VERSION}
          export SEEKER_AGENT_NAME=${AGENT}
          export MAVEN_OPTS=-javaagent:seeker/seeker-agent.jar

          serverMessage=$(/tmp/serverStart.sh --startCmd="${SERVER_START}" --startedString="${SERVER_STRING}" --project="${PROJECT}" --timeout="60s" &)
          if [[ $serverMessage == ?(-)+([0-9]) ]]; then #Check if value passed back is numeric (PID) or string (Error message).
            echo "Running IAST Tests"

            testRunID=$(curl -X 'POST' "${SEEKER_SERVER_URL}/rest/api/latest/testruns" -H 'accept: application/json' -H 'Content-Type: application/x-www-form-urlencoded' -H "Authorization: ${SEEKER_TOKEN}" -d "type=AUTO_TRIAGE&statusKey=FIXED&projectKey=${SEEKER_PROJECT_KEY}" | jq -r ".[]".key)
            echo "Run ID is : "$testRunID

            # Give Seeker some time to do it's stuff; API collation, testing etc.
            sleep ${SEEKER_RUN_TIME}

            testResponse=$(curl -X 'PUT' "${SEEKER_SERVER_URL}/rest/api/latest/testruns/$testRunID/close" -H 'accept: application/json' -H 'Content-Type: application/x-www-form-urlencoded' -H "Authorization: ${SEEKER_TOKEN}" -d 'completed=true')
            echo "Finished Testing. [$testResponse]"

            kill $serverMessage
          else
            echo $serverMessage
            return 1
          fi
        '''
      }
    }


    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
  }
}

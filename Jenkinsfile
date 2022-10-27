pipeline {
  agent any

  environment {
    APPLICATION = 'MJ-test-projects'
    PROJECT = 'juice-shop'
    VERSION = '1.0'
    BRANCH = 'main'
    POLARIS_ACCESS_TOKEN = credentials('POLARIS_TOKEN')
    CODEDX_TOKEN = credentials('CODEDX_TOKEN')
    SEEKER_TOKEN = credentials('SEEKER_TOKEN')
    SERVER_START = "npm run"
    SEEKER_STIFF = "java -javaagent:seeker/seeker-agent.jar -jar target/jhipster-sample-application-0.0.1-SNAPSHOT.jar"
    SERVER_STRING = "Application 'jhipsterSampleApplication' is running!"
    SERVER_WORKINGDIR = ""
    SEEKER_RUN_TIME = 180
    SEEKER_PROJECT_KEY = 'jshop'
    WORKSPACE_TMP = '/tmp'
  }

  stages{
    stage('NPM Install') {
      steps {
        sh 'npm install'
      }
    }

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

    stage('polaris') {
      steps {
        sh '''
          BRIDGE=https://sig-repo.synopsys.com/artifactory/bds-integrations-release/com/synopsys/integration/synopsys-action/0.1.72/ci-package-0.1.72-linux64.zip
          BRIDGE_POLARIS_SERVERURL=$POLARIS_SERVER_URL
          BRIDGE_POLARIS_ACCESSTOKEN=$POLARIS_ACCESS_TOKEN
          BRIDGE_POLARIS_APPLICATION_NAME=$APPLICATION
          BRIDGE_POLARIS_PROJECT_NAME=$PROJECT

          curl -fLsS -o $WORKSPACE_TMP/bridge.zip $BRIDGE
          unzip -qo -d $WORKSPACE_TMP/bridge $WORKSPACE_TMP/bridge.zip
          $WORKSPACE_TMP/bridge/bridge --stage polaris polaris.assessment.types='["SAST","SCA"]'
        '''
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}

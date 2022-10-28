pipeline {
  agent any

  environment {
    APPLICATION = 'MJ-test-projects'
    PROJECT = 'juice-shop'
    VERSION = '1.0'
    BRANCH = 'main'
    POLARIS_ACCESS_TOKEN = credentials('POLARIS_TOKEN')
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

    stage('Polaris') {
      steps {
        withCredentials([string(credentialsId: 'POLARIS_ACCESS_TOKEN', variable: 'BRIDGE_POLARIS_ACCESSTOKEN')]) {
          script {
            status = sh returnStatus: true, script: """
              export BRIDGE=https://sig-repo.synopsys.com/artifactory/bds-integrations-release/com/synopsys/integration/synopsys-action/0.1.72/ci-package-0.1.72-linux64.zip
              export BRIDGE_POLARIS_ACCESSTOKEN=$POLARIS_ACCESS_TOKEN
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

    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }
  }
}

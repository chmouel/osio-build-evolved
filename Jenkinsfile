def jobTimeOutHour = 1
def approvalTimeOutMinutes = 30

def askForInput() {
  def proceedMessage = """Would you like to promote to the next environment?
          """
  try {
    timeout(time: approvalTimeOutMinutes, unit: 'MINUTES') {
      input id: 'Proceed', message: "\n${proceedMessage}"
    }
  } catch (err) {
    throw err
  }
}

def deploy(_environ) {
  environ = "-"  + _environ
  // Populating istag to stage project
  try {
    sh "JSON=\$(oc get -o json is/${APPLICATION_NAME} -n ${TARGET_USER}${environ});oc delete is/${APPLICATION_NAME} -n ${TARGET_USER}${environ} && echo \$JSON|oc create -n ${TARGET_USER}${environ} -f -;oc get istag -n ${TARGET_USER}${environ}"
  } catch (err) {
    error "Error running OpenShift command ${err}"
  }

  openshiftDeploy(deploymentConfig: '${APPLICATION_NAME}', namespace: '${TARGET_USER}' + environ)

  try {
    ROUTE_PREVIEW = sh (
      script: "oc get route -n ${TARGET_USER}${environ} ${APPLICATION_NAME} --template 'http://{{.spec.host}}'",
      returnStdout: true
    ).trim()
    echo _environ.capitalize() + " URL: ${ROUTE_PREVIEW}"
  } catch (err) {
    error "Error running OpenShift command ${err}"
  }
}

try {
  timestamps{
    timeout(time: jobTimeOutHour, unit: 'HOURS') {
      node('nodejs') {
        stage('Build') {
          openshiftBuild(buildConfig: '${APPLICATION_NAME}', showBuildLogs: 'true')
        }

        stage('Deploy to staging') {
          deploy("stage")
          askForInput()
        }

        stage('Deploy to production') {
          deploy("run")
        }
      }
    }
  }
} catch (err) {
  echo "in catch block"
  echo "Caught: ${err}"
  currentBuild.result = 'FAILURE'
  throw err
}

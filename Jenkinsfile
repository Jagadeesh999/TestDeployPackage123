pipeline {
    // FIX #10: Label a dedicated Windows agent that has SoftwareAG installed.
    //          Change 'sag-windows-agent' to match your actual Jenkins node label.
    agent { label 'sag-windows-agent' }

    environment {
        SAG_HOME     = 'C:/SoftwareAG11'

        // FIX #6: JAVA_HOME and PATH kept for clarity but ANT_BIN / DEPLOYER_BIN
        //         use full paths, so PATH manipulation is a safety net only.
        JAVA_HOME    = "${SAG_HOME}/jvm/jvm"
        PATH         = "${JAVA_HOME}/bin;${env.PATH}"

        ABE_HOME     = "${SAG_HOME}/common/AssetBuildEnvironment"
        ANT_BIN      = "${SAG_HOME}/common/lib/ant/bin/ant"
        DEPLOYER_BIN = "${SAG_HOME}/IntegrationServer/instances/default/packages/WmDeployer/bin"

        // FIX #7: Derive ABE output dir from SAG_HOME instead of hardcoding it.
        ABE_OUTPUT   = "${SAG_HOME}/ABE_Output"

        // Names must match DeployerSpec XML exactly (see FIX #3 below).
        DEPLOYER_PROJECT   = 'ACDL_TestDeployPackage'
        DEPLOYER_CANDIDATE = 'myDeployment'
        DEPLOYER_HOST      = 'localhost'
        DEPLOYER_PORT      = '5555'
    }

    stages {

        // ----------------------------------------------------------------
        stage('Checkout Source') {
        // ----------------------------------------------------------------
            steps {
                echo 'Pulling code from GitHub (main branch)...'
                // FIX #9: Removed deprecated GitSCM parameters; using git shorthand.
                git branch: 'main',
                    url: 'https://github.com/Jagadeesh999/TestDeployPackage123.git'
            }
        }

        // ----------------------------------------------------------------
        stage('Build (ABE)') {
        // ----------------------------------------------------------------
            steps {
                echo 'Running Asset Build Environment (ABE)...'
                // FIX #7: Use ABE_OUTPUT env var instead of hardcoded path.
                bat """
                    "${ANT_BIN}" ^
                        -f "${ABE_HOME}/master_build/build.xml" ^
                        -Dbuild.source.dir="${WORKSPACE}" ^
                        -Dbuild.output.dir="${ABE_OUTPUT}" ^
                        -Denable.build.IS=true
                """
            }
        }

        // ----------------------------------------------------------------
        stage('Project Setup') {
        // ----------------------------------------------------------------
            steps {
                echo 'Running Project Automator to register project in Deployer...'
                // FIX #1: Uncommented projectAutomator.bat — this MUST run before Deploy
                //         so that the project exists in Deployer when --deploy is called.
                bat "\"${DEPLOYER_BIN}\\projectAutomator.bat\" \"${WORKSPACE}\\ProjectAutomator.xml\""
            }
        }

        // ----------------------------------------------------------------
        stage('Deploy') {
        // ----------------------------------------------------------------
            steps {
                echo 'Deploying package via WmDeployer...'
                // FIX #1: Removed projectAutomator.bat from here — it belongs in Project Setup.
                // FIX #2: Credentials pulled from Jenkins credential store, not plaintext.
                // FIX #3: Project name (-project) and candidate (-dc) now match the
                //         DeployerSpec XML: ACDL_TestDeployPackage / myDeployment.
                withCredentials([usernamePassword(
                        credentialsId: 'is-admin-creds',   // <-- set this up in Jenkins
                        usernameVariable: 'IS_USER',
                        passwordVariable: 'IS_PWD')]) {
                    bat """
                        "${DEPLOYER_BIN}\\Deployer.bat" ^
                            --deploy ^
                            -project "${DEPLOYER_PROJECT}" ^
                            -dc "${DEPLOYER_CANDIDATE}" ^
                            -host "${DEPLOYER_HOST}" ^
                            -port "${DEPLOYER_PORT}" ^
                            -user "%IS_USER%" ^
                            -pwd "%IS_PWD%"
                    """
                }
            }
        }
    }

    // FIX #8: Added post block for failure notifications and workspace cleanup.
    post {
        success {
            echo "Deployment of '${DEPLOYER_PROJECT}' completed successfully."
        }
        failure {
            echo "Pipeline failed — check console output above for details."
            // Uncomment to send email on failure:
            // mail to: 'your-team@example.com',
            //      subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            //      body: "See: ${env.BUILD_URL}"
        }
        cleanup {
            // Wipe workspace after every run to avoid stale ABE artefacts.
            cleanWs()
        }
    }
}

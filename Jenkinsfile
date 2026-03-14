pipeline {
    agent any

    environment {
        // Your specific local path
        SAG_HOME = 'C:/SoftwareAG11'
        
        // Tool Paths
        ABE_HOME = "${env.SAG_HOME}/common/AssetBuildEnvironment"
        ANT_BIN  = "${env.SAG_HOME}/common/lib/ant/bin/ant"
        DEPLOYER_BIN = "${env.SAG_HOME}/IntegrationServer/instances/default/packages/WmDeployer/bin"
        
        // Project Details
        REPO_URL = 'https://github.com'
        PROJECT_NAME = 'TestDeployProject'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning ${env.REPO_URL}..."
                git branch: 'main', url: "${env.REPO_URL}"
            }
        }

        stage('Build (ABE)') {
            steps {
                echo 'Packaging assets using ABE...'
                // Point ABE to the current Jenkins workspace for source
                bat """
                ${env.ANT_BIN} -f ${env.ABE_HOME}/master_build/build.xml \
                -Dbuild.source.dir=${WORKSPACE} \
                -Dbuild.output.dir=${env.SAG_HOME}/ABE_Output \
                -Denable.build.IS=true
                """
            }
        }

        stage('Project Automator') {
            steps {
                echo 'Creating/Updating Deployer Project...'
                // Ensure your ProjectAutomator.xml is in the root of your GitHub repo
                bat "${env.DEPLOYER_BIN}/projectAutomator.bat -file ${WORKSPACE}/ProjectAutomator.xml"
            }
        }

        stage('Deploy') {
            steps {
                echo 'Executing Deployment...'
                // Runs the deployment candidate defined in your XML
                bat "${env.DEPLOYER_BIN}/Deployer.bat --deploy -project ${env.PROJECT_NAME} -dc MyCandidate -host localhost -port 5555 -user Administrator -pwd manage"
            }
        }
    }
}

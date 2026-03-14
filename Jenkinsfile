pipeline {
    agent any

  environment {
    SAG_HOME = 'C:/SoftwareAG11'
    // Point to the JVM bundled with webMethods
    JAVA_HOME = "${env.SAG_HOME}/jvm/jvm" 
    
    // Add Java and Ant to the PATH for this session
    PATH = "${env.JAVA_HOME}/bin;${env.SAG_HOME}/common/lib/ant/bin;${env.PATH}"
    
    ABE_HOME = "${env.SAG_HOME}/common/AssetBuildEnvironment"
    ANT_BIN  = "${env.SAG_HOME}/common/lib/ant/bin/ant"
    DEPLOYER_BIN = "${env.SAG_HOME}/IntegrationServer/instances/default/packages/WmDeployer/bin"
}


    stages {
stage('Checkout Source') {
    steps {
        echo 'Pulling code from GitHub (main branch)...'
        checkout([$class: 'GitSCM', 
            branches: [[name: '*/main']], 
            doGenerateSubmoduleConfigurations: false, 
            extensions: [], 
            submoduleCfg: [], 
            userRemoteConfigs: [[url: 'https://github.com/Jagadeesh999/TestDeployPackage123.git']]
        ])
    }
}


stage('Build (ABE)') {
    steps {
        // Use double quotes for paths with spaces if necessary
        bat "${env.ANT_BIN} -f ${env.ABE_HOME}/master_build/build.xml -Dbuild.source.dir=${WORKSPACE} -Dbuild.output.dir=C:/SoftwareAG11/ABE_Output -Denable.build.IS=true"
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

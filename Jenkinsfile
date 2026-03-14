pipeline {
    agent any
    environment {
        SAG_HOME = 'C:/SoftwareAG11'
        // Point to the bundled JVM so 'java.exe' is found
        JAVA_HOME = "${env.SAG_HOME}/jvm/jvm"
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
        // ABE will now package 'TestDeployPackage' found in the workspace
        bat "${env.ANT_BIN} -f ${env.ABE_HOME}/master_build/build.xml -Dbuild.source.dir=${WORKSPACE} -Dbuild.output.dir=C:/SoftwareAG11/ABE_Output -Denable.build.IS=true"
            }
        }



stage('Project Setup') {
    steps {
        echo 'Running Project Automator...'
        // Remove '-file' and wrap the path in double quotes with backslashes
       // bat "${env.DEPLOYER_BIN}\\projectAutomator.bat \"${WORKSPACE}\\ProjectAutomator.xml\""
    }
}


        stage('Deploy') {
            steps {
                // Remove -file flag and use direct path to XML
                bat "${env.DEPLOYER_BIN}\\projectAutomator.bat \"${WORKSPACE}\\ProjectAutomator.xml\""
                // Execute actual push
                bat "${env.DEPLOYER_BIN}\\Deployer.bat --deploy -project TestDeployProject -dc MyCandidate -host localhost -port 5555 -user Administrator -pwd manage"
            }
        }
    }
}

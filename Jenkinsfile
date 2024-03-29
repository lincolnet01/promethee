pipeline {
    agent any

    parameters{
        choice( choices:['init','none'], description: 'Axelor Sources Expectations', name: 'AXELOR_CODE')
    // choice( choices:['clone','pull'], description: 'Appolo Sources Expectations', name: 'APPOLO_CODE')
    }
    environment {
        // GRADLE_OPTS = "-Dorg.gradle.daemon=false"
        AXELOR_SOURCES_DIR = "axelor-sources-code"
        // CUSTOM_PROJECT_DIR = "axelor-promethee"
        PROJECT_NAME="promethee"
        CICD_WORKBENCH = "promethee/ci-cd"
        CICD_ENV="dev"
        SSH_USER = "root" //Utile en production on aura besoin de se connecter en SSH etc...
        SSH_SERVER_IP = ""
        SSH_PASSWORD = ""
        DB_PWD = "${PROJECT_NAME}@2023!${CICD_ENV}"
        APP_ENC_PWD = "CiCd@Appolo@2023!"

    }
    stages {
        stage('Axelor: récuperer le code') {
            steps{
                sh 'mkdir -p ${AXELOR_SOURCES_DIR}'
                checkout scmGit(
                    branches: [[name: '*/master']], 
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "${AXELOR_SOURCES_DIR}"]], 
                    userRemoteConfigs: [[ url: 'https://github.com/axelor/open-suite-webapp.git']]
                )
            }
        }
        stage('Axelor : initialiser les sous-modules'){
            when{
                expression { return env.AXELOR_CODE == 'init'}
            }
            steps{
          
                sh '''
                sed -e 's|git@github.com:|https://github.com/|' -i ${AXELOR_SOURCES_DIR}/.gitmodules
                cd ${AXELOR_SOURCES_DIR}
                git checkout master
                git submodule sync
                git submodule init
                git submodule update
                git submodule foreach git checkout master
                git submodule foreach git pull origin master
                ls -al modules
                cd ..
                '''
            }
   
        }

        stage('Appolo: récuperer les sources du projet'){
  
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: "${AXELOR_SOURCES_DIR}/modules/axelor-open-suite/axelor-${PROJECT_NAME}"]], userRemoteConfigs: [[credentialsId: 'cicd.appolo-consulting.com', url: 'http://cicd.appolo-consulting.com/prod-team/promethee.git']])
                sh '''
                mkdir -p ${CICD_WORKBENCH}/${CICD_ENV}/apps
                mkdir -p ${CICD_WORKBENCH}/${CICD_ENV}/axelor
                mkdir -p ${CICD_WORKBENCH}/${CICD_ENV}/proxy
                mkdir -p ${CICD_WORKBENCH}/${CICD_ENV}/ci
                '''
            }
        }

        stage('Axelor: Construire le fichier .war'){

            steps{

                sh '''
                cd ${AXELOR_SOURCES_DIR}
                ./gradlew clean build -x test 
                cd ..
                cp ${AXELOR_SOURCES_DIR}/build/libs/*.war ${CICD_WORKBENCH}/${CICD_ENV}/apps/ROOT.war
                '''
            }

        }
    
    }
    options {
        skipDefaultCheckout()
    }
}

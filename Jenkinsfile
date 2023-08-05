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
                ls -al ${AXELOR_SOURCES_DIR}/modules
                cd ..
                '''
            }
   
        }

        stage('Appolo: récuperer les sources du projet'){
  
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '$AXELOR_SOURCES_DIR/modules/axelor-open-suite/axelor-$PROJECT_NAME']], userRemoteConfigs: [[credentialsId: 'cicd.appolo-consulting.com', url: 'http://cicd.appolo-consulting.com/prod-team/promethee.git']])
                sh '''
                mkdir -p ${CICD_WORKBENCH}/${CICD_ENV}/{apps, axelor,proxy,ci}
                '''
            
            }
        }

        stage('Axelor: Construire le fichier .war'){

            steps{

                sh '''
                cd ${AXELOR_SOURCES_DIR}
                ./gradlew clean build -x test 
                cp ${AXELOR_SOURCES_DIR}/build/libs/*.war ${CICD_WORKBENCH}/${CICD_ENV}/apps/ROOT.war
                cd ..
                '''
            }

        }
        stage('Docker: Configurer les fichiers necessaires pour construire les conteneurs'){

            steps{
                script{
                    def db_pwd = ('0'..'z').shuffled().take(10).join()
                    def app_enc_pwd= "CiCd@Appolo@2023!"
                }
                checkout scmGit(branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '$CICD_WORKBENCH/$CICD_ENV/ci']], userRemoteConfigs: [[credentialsId: 'cicd.appolo-consulting.com', url: 'http://cicd.appolo-consulting.com/sysadmin/cicd.git']])
   
                sh '''
                cp ${CICD_WORKBENCH}/${CICD_ENV}/ci/docker-compose.yml ${CICD_WORKBENCH}/${CICD_ENV}/docker-compose.yml
                cp ${CICD_WORKBENCH}/${CICD_ENV}/ci/.env ${CICD_WORKBENCH}/${CICD_ENV}/.env
                cp ${CICD_WORKBENCH}/${CICD_ENV}/ci/axelor/axelor-config.properties $CICD_WORKBENCH/$CICD_ENV/axelor/axelor-config.properties
                sed -e 's|db_name:|${PROJECT_NAME}-db|' -i ${CICD_WORKBENCH}/${CICD_ENV}/.env
                sed -e 's|db_user:|${PROJECT_NAME}-app|' -i ${CICD_WORKBENCH}/${CICD_ENV}/.env
                sed -e 's|db_password:|${db_pwd}|' -i ${CICD_WORKBENCH}/${CICD_ENV}/.env
                sed -e 's|encryption_password:|${app_enc_pwd}|' -i ${CICD_WORKBENCH}/${CICD_ENV}/.env
                sed -e 's|project_name:|${PROJECT_NAME}|' -i ${CICD_WORKBENCH}/${CICD_ENV}/axelor/axelor-config.properties
                sed -e 's|project_env:|${CICD_ENV}|' -i ${CICD_WORKBENCH}/${CICD_ENV}/axelor/axelor-config.properties
                sed -e 's|project_name:|${PROJECT_NAME}|' -i ${CICD_WORKBENCH}/${CICD_ENV}/docker-compose.yml
                sed -e 's|project_env:|${CICD_ENV}|' -i ${CICD_WORKBENCH}/${CICD_ENV}/docker-compose.yml
                cd ${CICD_WORKBENCH}/${CICD_ENV}/
                docker compose down
                docker compose up -d
                cd ..
                '''
            }
        }
    
    }
    options {
        skipDefaultCheckout()
    }
}

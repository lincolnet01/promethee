pipeline {
    agent any
    // agent {
        // docker {
        //     image 'gradle:jdk11'
        //     args '-v $HOME/axelor-sources:/axelor-sources'
        // }
    // }
    environment {
        // GRADLE_OPTS = "-Dorg.gradle.daemon=false"
        AXELOR_SOURCES_DIR = "~/axelor-sources"
        // CUSTOM_PROJECT_DIR = "axelor-aletheia"
        PROJECT_NAME="aletheia"
        CICD_WORKBENCH = "~/aletheia/ci-cd"
        CICD_ENV="dev"
        SSH_USER = "root" //Utile en production on aura besoin de se connecter en SSH etc...
        SSH_SERVER_IP = ""
        SSH_PASSWORD = ""

    }
    stages {
        stage('Clone Official Repository Of Axelor'){
            steps{
                sh '''
                mkdir -p $AXELOR_SOURCES_DIR
                git clone https://github.com/axelor/open-suite-webapp.git $AXELOR_SOURCES_DIR
                sed -e 's|git@github.com:|https://github.com/|' -i $AXELOR_SOURCES_DIR/.gitmodules
                git checkout master
                git submodule sync
                git submodule init
                git submodule update
                git submodule foreach git checkout master
                git submodule foreach git pull origin master
                ls -al $AXELOR_SOURCES_DIR/modules
                '''
            }
   
        }
        stage('Pull Custom Code from APPOLO CONSULTING'){
            steps{
                checkout scmGit(branches: [[name: '*/master']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '$AXELOR_SOURCES_DIR/modules/axelor-open-suite/axelor-$PROJECT_NAME']], userRemoteConfigs: [[credentialsId: 'cicd.appolo-consulting.com', url: 'http://cicd.appolo-consulting.com/prod-team/promethee.git']])
            }
        }
        stage('Build .war'){
            steps{
                sh '''
                mkdir -p $CICD_WORKBENCH/$CICD_ENV/{apps, axelor,proxy,ci}
                $AXELOR_SOURCES_DIR/gradlew clean build -x test
                ls -l $AXELOR_SOURCES_DIR/build/libs
                cp $AXELOR_SOURCES_DIR/build/libs/*.war $CICD_WORKBENCH/$CICD_ENV/apps/ROOT.war
                '''
            }
  
        }
        stage('Manage Docker Layer'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '$CICD_WORKBENCH/$CICD_ENV/ci']], userRemoteConfigs: [[credentialsId: 'cicd.appolo-consulting.com', url: 'http://cicd.appolo-consulting.com/sysadmin/cicd.git']])
                def db_pwd = ('0'..'z').shuffled().take(10).join()
                def app_enc_pwd= "CiCd@Appolo@2023!"
                sh '''
                cp $CICD_WORKBENCH/$CICD_ENV/ci/docker-compose.yml $CICD_WORKBENCH/$CICD_ENV/docker-compose.yml
                cp $CICD_WORKBENCH/$CICD_ENV/ci/.env $CICD_WORKBENCH/$CICD_ENV/.env
                cp $CICD_WORKBENCH/$CICD_ENV/ci/axelor/axelor-config.properties $CICD_WORKBENCH/$CICD_ENV/axelor/axelor-config.properties
                sed -e 's|db_name:|$PROJECT_NAME-db|' -i $CICD_WORKBENCH/$CICD_ENV/.env
                sed -e 's|db_user:|$PROJECT_NAME-app|' -i $CICD_WORKBENCH/$CICD_ENV/.env
                sed -e 's|db_password:|$db_pwd|' -i $CICD_WORKBENCH/$CICD_ENV/.env
                sed -e 's|encryption_password:|$app_enc_pwd|' -i $CICD_WORKBENCH/$CICD_ENV/.env
                sed -e 's|project_name:|$PROJECT_NAME|' -i $CICD_WORKBENCH/$CICD_ENV/axelor/axelor-config.properties
                sed -e 's|project_env:|$CICD_ENV|' -i $CICD_WORKBENCH/$CICD_ENV/axelor/axelor-config.properties
                sed -e 's|project_name:|$PROJECT_NAME|' -i $CICD_WORKBENCH/$CICD_ENV/docker-compose.yml
                sed -e 's|project_env:|$CICD_ENV|' -i $CICD_WORKBENCH/$CICD_ENV/docker-compose.yml
                '''
            }
        }
    
    }
    options {
        skipDefaultCheckout()
    }
}
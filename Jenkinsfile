pipeline {
    agent {
        docker {
            image 'gradle:jdk11'
            args '-v $HOME/axelor-sources:~/axelor-sources'
        }
    }
    environment {
        GRADLE_OPTS = "-Dorg.gradle.daemon=false"
        AXELOR_SOURCES_DIR = "~/axelor-sources"
        CUSTOM_PROJECT_NAME = "axelor-promethee"
        CICD_WORKBENCH = "~/ci-cd"
        CICD_ENV="dev"
        SSH_USER = "root" //Utile en production on aura besoin de se connecter en SSH etc...
        SSH_SERVER_IP = ""
        SSH_PASSWORD = ""

    }
    stages {
        stage('Clone Official Repository Of Axelor'){
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
        stage('Pull Custom Code from APPOLO CONSULTING'){
            checkout([$class: 'GitSCM',
                branches: [[name: '*/master' ]],
                extensions: [
                                [$class: 'RelativeTargetDirectory', relativeTargetDir: '$AXELOR_SOURCES_DIR/modules/axelor-open-suite/$CUSTOM_PROJECT_NAME'],
                                [$class: 'GitLFSPull'],
                                [$class: 'CheckoutOption', timeout: 20],
                                [$class: 'CloneOption',
                                    depth: 3,
                                    noTags: false,
                                    reference: '$AXELOR_SOURCES_DIR/modules/axelor-open-suite/$CUSTOM_PROJECT_NAME',
                                    shallow: true,
                                    timeout: 120],
                                [$class: 'SubmoduleOption', depth: 5, disableSubmodules: false, parentCredentials: true, recursiveSubmodules: true, reference: '', shallow: true, trackingSubmodules: true]
                            ],
                userRemoteConfigs: [[
                    url: 'http://cicd.appolo-consulting.com/prod-team/promethee.git',
                    credentialsId: 'cicd.appolo-consulting.com'
                ]]
            ])
        }
        stage('Build .war'){
            sh '''
            mkdir -p $CICD_WORKBENCH/$CICD_ENV/{build,ci}
            $AXELOR_SOURCES_DIR/gradlew clean build -x test
            ls -l $AXELOR_SOURCES_DIR/build/libs
            cp $AXELOR_SOURCES_DIR/build/libs/*.war $CICD_WORKBENCH/$CICD_ENV/build/ROOT.war
            '''
        }
        stage('Manage Docker Layer'){
            echo 'Manipule le docker-compose.yml ici pour lancer l''instance axelor'
        }
    
    }
    options {
        skipDefaultCheckout()
    }
}

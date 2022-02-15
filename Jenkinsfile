def buildList = [:]

pipeline {

    agent any

    stages {

        stage('Prepare') {
            steps {
                script {
                    def changePathList = []

                    for (changeSet in currentBuild.changeSets) {
                        for (entry in changeSet.getItems()) {
                            for (file in entry.getAffectedFiles()) {
                                changePathList.add(file.getPath())
                            }
                        }
                    }

                    def pathRegex = /(\w*)\/.*/

                    for (changePath in changePathList) {
                        if (!(changePath ==~ pathRegex)) continue

                        def rootPath = (changePath =~ pathRegex)[0][1]

                        if (
                            buildList.containsKey(rootPath) ||
                            !fileExists(rootPath + '/service.json') ||
                            !fileExists(rootPath + '/Dockerfile')
                        ) continue

                        def service = readJSON(file: rootPath + '/service.json')
                        buildList.put(rootPath, service)
                    }
                }
            }
        }

        stage('Build') {

            steps {
                echo 'building the aplication...'

                script {
                    parallel buildList.collectEntries {
                        app, service -> [app, {
                            stage ("Building $app app") {
                                sh """
                                    docker build -t $app:$BUILD_NUMBER ./$app
                                """
                            }
                        }]
                    }
                }
            }
        }

         stage('Deploy') {

            steps {
                echo 'deploying the aplication...'

                script {
                    parallel buildList.collectEntries{
                        app, service -> [app, {
                            stage ("Deploying $app app") {
                                def inspectExitCode = sh(script: "docker service inspect $service.name", returnStatus: true)

                                if (inspectExitCode == 0) {
                                    sh """
                                        docker service update \
                                        --replicas $service.replicas \
                                        --image $app:$BUILD_NUMBER \
                                        $service.name
                                    """
                                } else {
                                    sh """
                                        docker service create \
                                        --name $service.name \
                                        --replicas $service.replicas \
                                        --publish $service.publish \
                                        $app:$BUILD_NUMBER
                                    """
                                }
                            }
                        }]
                    }
                }
            }
        }
    }
}

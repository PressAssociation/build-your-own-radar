#!/usr/bin/env groovy

@Library('pa-shared')
import java.lang.Object

properties([[$class: 'ParametersDefinitionProperty', parameterDefinitions:
        [[$class: 'StringParameterDefinition', defaultValue: 'build-your-own-radar', description: 'Build your own radar', name: 'outputContainerName'],
         [$class: 'StringParameterDefinition', defaultValue: 'pa-registry.transform.pacpservices.net', description: 'PA Registry', name: 'dockerRegistryHost']
        ]]])

String tag

node() {
    try {
        fallibleStage("Checkouts") {
            checkout scm
            switch(BRANCH_NAME) {
                case "master":
                    tag = "${getProjectVersion()}"
                    break
                case "develop":
                    tag = "${getProjectVersion()}-${getCommitID()}"
                    break
                default:
                    tag = BRANCH_NAME.replace("/", "-")
                    break
            }
            echo "BUILD VERSION DETERMINED AS : ${tag}"
        }

        fallibleStage("Build") {
            nodejs(nodeJSInstallationName: 'Node8', configId: 'basenpmrc') {
                echo "Beginning build with npm..."
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install'
            }
        }

        fallibleStage("Test") {
            nodejs(nodeJSInstallationName: 'Node8', configId: 'basenpmrc') {
                sh "npm run test"
            }
        }

        switch (BRANCH_NAME) {
            case "develop":
                fallibleStage("Build Docker image") {
                    docker.withRegistry("https://${params.dockerRegistryHost}", '13efcab7-d472-4c8e-836e-d60b26dc754c') {
                        buildDockerImage(params.dockerRegistryHost, params.outputContainerName, tag, ["VERSION": tag])
                    }
                }
                break
            case "master":
                fallibleStage("Build Docker image") {
                    docker.withRegistry("https://${params.dockerRegistryHost}", '13efcab7-d472-4c8e-836e-d60b26dc754c') {
                        buildDockerImage(params.dockerRegistryHost, params.outputContainerName, tag)
                    }
                }
                break
            default:
              if (BRANCH_NAME.matches("feature/.*")) {
                    fallibleStage("Build Docker image") {
                        docker.withRegistry("https://${params.dockerRegistryHost}", '13efcab7-d472-4c8e-836e-d60b26dc754c') {
                            buildDockerImage(params.dockerRegistryHost, params.outputContainerName, tag, ["VERSION": tag], false, false)
                        }
                    }
               }
               break
        }

    }
    finally {
        notifyResults()
        thisRegistry = null // clean this up, as JsonSlurper objects are non-serializable
        cleanWs()
    }
}

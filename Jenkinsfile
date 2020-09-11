pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "master" || env.BRANCH_NAME == "develop" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "master" ? "goapp" : "goapp-develop"}"
        VERSION = "1.0"
        REGISTRY = "bsakweson/goapp"
        REGISTRY_CREDENTIAL = "dockerhub-bsakweson"
    }
    agent {
        kubernetes {
            label mignons
            defaultContainer "jnlp"
            yamlFile "build.yaml"
        }
    }
    stages {
        stage("Build") {
            steps {
                container("golang") {
                    sh "go build"
                }
            }
        }

        stage("Test") {
            steps {
                container("golang") {
                    sh " go test -v -run PizzasHandler"
                }
            }
        }
        stage("Docker Build") {
            when {
                environment name: "DEPLOY", value: "true"
            }
            steps {
                container("docker") {
                    sh "docker build -t ${REGISTRY}:${VERSION} ."
                }
            }
        }
        stage("Docker Publish") {
            when {
                environment name: "DEPLOY", value: "true"
            }
            steps {
                container("docker") {
                    withDockerRegistry([credentialsId: "${REGISTRY_CREDENTIAL}", url: ""]) {
                        sh "docker push ${REGISTRY}:${VERSION}"
                    }
                }
            }
        }
        stage("Kubernetes Deploy") {
            when {
                environment name: "DEPLOY", value: "true"
            }
            steps {
                container("helm") {
                    sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} ./goapp"
                }
            }
        }
    }
}
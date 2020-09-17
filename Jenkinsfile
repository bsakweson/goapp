// This is a test
def label = "mignon-${UUID.randomUUID().toString()}"
def namespace

podTemplate(label:  label, containers: [
    containerTemplate(name: "golang", image: "golang:1.14", command: "cat", ttyEnabled: true),
    containerTemplate(name: "docker", image: "docker:19.03", command: "cat", ttyEnabled: true),
    containerTemplate(name: "helm", image: "lachlanevenson/k8s-helm:v3.3.1", command: "cat", ttyEnabled: true),
    containerTemplate(name: "kubectl", image: "lachlanevenson/k8s-kubectl:v1.8.8", command: "cat", ttyEnabled: true),
],
volumes: [
    hostPathVolume(mountPath: "/var/run/docker.sock", hostPath: "/var/run/docker.sock")
]) {
    node(label) {
        def myRepo = checkout scm
        def gitCommit = myRepo.GIT_COMMIT
        def gitBranch = myRepo.GIT_BRANCH
        def shortGitCommit = "${gitCommit[0..10]}"
        def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)

        stage("Build App"){
            container("golang"){
                sh "go build"
            }
        }
        stage("Test App"){
            try{
                container("golang"){
                    sh "go test -v -run PizzasHandler"
                }
            }catch(exc) {
                println "Failed to test - ${currentBuild.fullDisplayName}"
                throw(exc)
            }
        }
        stage("Create Docker Image") {
            container("docker") {
                withCredentials([[$class: "UsernamePasswordMultiBinding",
                credentialsId: "dockerhub-bsakweson",
                usernameVariable: "DOCKER_HUB_USER",
                passwordVariable: "DOCKER_HUB_PASSWORD"]]) {
                    sh """
                    docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
                    docker build -t bsakweson/goapp:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage("Push Image To Registry") {
            container("docker") {
                sh "docker push bsakweson/goapp:${BUILD_NUMBER}"
            }
        }


        stage("Helm Deploy") {
            if(gitBranch == "master"){
                namespace = "qa"
            }else if(gitBranch == "develop") {
                namespace = "development"
            }else if(gitBranch == "release") {
                namespace = "production"
            } else {
                namespace = "development"
            } 

            container("helm"){
                sh "helm upgrade goapp-${namespace} ./goapp -n ${namespace} -i --wait --set name=goapp-${namespace} --set image.tag=${BUILD_NUMBER}"
            }
        }
    }
}
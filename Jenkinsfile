// Configuration variables
github_org             = "samsung-cnct"
quay_org               = "samsung_cnct"
publish_branch         = "master"
image_tag              = "${env.RELEASE_VERSION}" != "null" ? "${env.RELEASE_VERSION}" : "latest"
project_name           = "solas-container"

podTemplate(label: "${project_name}", containers: [
    containerTemplate(name: 'jnlp', image: "quay.io/${quay_org}/custom-jnlp:0.1", args: '${computer.jnlpmac} ${computer.name}'),
    containerTemplate(name: 'docker', image: 'docker:17.09.0-ce-git', command: 'cat', ttyEnabled: true)
  ], volumes: [
    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
    hostPathVolume(hostPath: '/var/lib/docker/scratch', mountPath: '/mnt/scratch'),
    secretVolume(mountPath: '/home/jenkins/.docker/', secretName: 'samsung-cnct-quay-robot-dockercfg')
  ]) {
    node("${project_name}") {
      customContainer('docker') {
        // add a docker rmi/docker purge/etc.
        stage('Checkout') {
          checkout scm
          // retrieve the URI used for checking out the source
          // this assumes one branch with one uri
          git_uri = scm.getRepositories()[0].getURIs()[0].toString()
          git_branch = scm.getBranches()[0].toString()
        }
        // build new version of kraken-tools image on 'docker' container
        stage('Build') {
          kubesh "docker version"
          kubesh "docker build -t ${project_name}:${env.JOB_BASE_NAME}.${env.BUILD_ID} ."
        }

        stage('Test') {
          kubesh "echo The test stage is empty"
        }

        // only push from master.   check that we are on samsung-cnct fork
        stage('Publish') {
          if (git_branch.contains(publish_branch) && git_uri.contains(github_org)) {
            kubesh "docker tag ${project_name}:${env.JOB_BASE_NAME}.${env.BUILD_ID} quay.io/${quay_org}/${project_name}:${image_tag}"
            kubesh "docker push quay.io/${quay_org}/${project_name}:${image_tag}"
          } else {
            echo "Not pushing to docker repo:\n    BRANCH_NAME='${env.BRANCH_NAME}'\n    GIT_BRANCH='${git_branch}'\n    git_uri='${git_uri}'"
          }
        }
      }
    }
  }

def kubesh(command) {
  if (env.CONTAINER_NAME) {
    if ((command instanceof String) || (command instanceof GString)) {
      command = kubectl(command)
    }

    if (command instanceof LinkedHashMap) {
      command["script"] = kubectl(command["script"])
    }
  }
  sh(command)
}

def kubectl(command) {
  "kubectl exec -i ${env.HOSTNAME} -c ${env.CONTAINER_NAME} -- /bin/sh -c 'cd ${env.WORKSPACE} && ${command}'"
}

def customContainer(String name, Closure body) {
  withEnv(["CONTAINER_NAME=$name"]) {
    body()
  }
}

// vi: ft=groovy

#!/usr/bin/env groovy

library 'defn/jenkins-kiki@main'

def config = [
  role: 'defn--hello',
  roleId: '7a87edd4-68d9-d7fb-974b-752f030c65b9',
  pipelineSecrets: [[
    path: 'kv/pipeline/defn--hello',
    secretValues: [
      [vaultKey: 'MEH1'],
      [vaultKey: 'MEH2']
    ]
  ]]
]

goreleaserMain(config) {
  stage('Style') {
    sh("make style")
  }

  stage('Test') {
    sh("make test")
  }

  stage('Test inside Docker') {
    docker.image("defn/jenkins").inside {
      sh """
        pwd
        uname -a
        id -a
        env | cut -d= -f1 | sort | xargs
        /env.sh go fmt; if test -n "\$(git status --porcelain)"; then git diff; exit 1; fi
      """
    }
  }

  if (env.TAG_NAME) {
    def image = "defn/hello:${env.GORELEASER_CURRENT_TAG.minus('v')}-amd64"
    withEnv(["DOCKER_IMAGE=${image}"]) {
      stage('Test Docker image') {
        sh("make docker")
      }
    }
  }
}

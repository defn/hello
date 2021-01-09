#!/usr/bin/env groovy

def kiki = library 'defn/jenkins-kiki@main'

def NM_DOCKER = 'defn/hello'
def VENDOR_PREFIX = ''
def NM_BINARY = 'hello'

def NM_PROJECT = 'defn/hello'
def NM_JOB= 'defn--hello'
def pipelineRoleId = '7a87edd4-68d9-d7fb-974b-752f030c65b9'

def githubSecrets = [
  [ 
    path: 'kv/jenkins/common',
    secretValues: [
      [vaultKey: 'GITHUB_TOKEN'],
      [vaultKey: 'DOCKER_USERNAME'],
      [vaultKey: 'DOCKER_PASSWORD']
    ]
  ]
]

def pipelineSecrets = [
  [ 
    path: 'kv/pipeline/' + NM_JOB,
    secretValues: [
      [vaultKey: 'MEH1'],
      [vaultKey: 'MEH2']
    ]
  ]
]

node() {
  checkout scm

  if (env.TAG_NAME == null) {
    env.GORELEASER_CURRENT_TAG = "0.${env.CHANGE_ID ?: 0}.${env.BUILD_ID}-${env.BUILD_TAG}"

    stage('Tag') {
      sh "git tag ${env.GORELEASER_CURRENT_TAG}"
    }
  }
  else {
    env.GORELEASER_CURRENT_TAG = env.TAG_NAME
  }

  withCredentials([[
    $class: 'VaultTokenCredentialBinding',
    credentialsId: 'VaultToken',
    vaultAddr: env.VAULT_ADDR ]]) {

    stage ('Secrets') {
      def PIPELINE_SECRET_ID= ''
      env.PIPELINE_SECRET_ID = sh(returnStdout: true, script: "./ci/build ${NM_JOB}").trim()

      def pipelineConfiguration = creds(pipelineRoleId, env.PIPELINE_SECRET_ID)

      withVault([vaultSecrets: pipelineSecrets, configuration: pipelineConfiguration]) {
        sh("env | grep MEH")
      }
    }

    stage('Tests') {
      sh "true"
    }

    meh({})

    withVault([vaultSecrets: githubSecrets]) {
      withEnv(["DOCKER_CONFIG=/tmp/docker/${env.BUILD_TAG}"]) {
        if (env.TAG_NAME) {
          stage('Release') {
            sh "install -d -m 0700 /tmp/docker"
            sh "install -d -m 0700 /tmp/docker/${env.BUILD_TAG}"
            sh "env | grep ^DOCKER_PASSWORD= | cut -d= -f2- | docker login --password-stdin --username ${DOCKER_USERNAME}"
            sh "/env.sh goreleaser release --rm-dist"
          }

          stage('Test Docker image') {
            sh "/env.sh docker run --rm --entrypoint /" + NM_BINARY + "  " + NM_DOCKER + ":" + VENDOR_PREFIX + "${env.GORELEASER_CURRENT_TAG.minus('v')}-amd64"
          }
        }
        else {
          stage('Build') {
            sh "/env.sh goreleaser build --rm-dist"
          }
        }
      }
    }

    stage('Cleanup') {
      sh "rm -rf /tmp/docker/${env.BUILD_TAG}"
    }
  }
}

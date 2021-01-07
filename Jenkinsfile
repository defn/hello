#!/usr/bin/env groovy

import hudson.util.Secret
import com.cloudbees.plugins.credentials.CredentialsScope
import com.datapipe.jenkins.vault.credentials.VaultTokenCredential

def NM_ROLE = 'pipeline'
def ID_ROLE = 'b45fcd66-6e60-3c2f-57e9-c0c5ecd59df2'

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
    path: 'kv/pipeline/defn/hello',
    secretValues: [
      [vaultKey: 'MEH1'],
      [vaultKey: 'MEH2']
    ]
  ]
]

node() {
  checkout scm

  env.GORELEASER_CURRENT_TAG = "0.${env.CHANGE_ID ?: 0}.${env.BUILD_ID}-${env.BUILD_TAG}"

  withCredentials([[
    $class: 'VaultTokenCredentialBinding',
    credentialsId: 'VaultToken',
    vaultAddr: env.VAULT_ADDR ]]) {

    stage ('Pipeline Secrets') {
      def PIPELINE_TOKEN = ''
      env.PIPELINE_TOKEN = sh(returnStdout: true, script: "./ci/build ${NM_ROLE} ${ID_ROLE}").trim()

      VaultTokenCredential pipelineCredential = new VaultTokenCredential(
        CredentialsScope.GLOBAL,
        'defn--hello-vault', 'defn--hello-vault',
        Secret.fromString(env.PIPELINE_TOKEN)
      )

      def pipelineConfiguration = [
        vaultCredential: pipelineCredential
      ]

      withVault([vaultSecrets: pipelineSecrets, configuration: pipelineConfiguration]) {
        sh "env | grep MEH"
      }
    }

    stage ('Tag') {
      sh "git tag ${env.GORELEASER_CURRENT_TAG}"
    }

    stage('Build') {
      withVault([vaultSecrets: githubSecrets]) {
        withEnv(["DOCKER_CONFIG=/tmp/${env.BUILD_TAG}"]) {
          sh "env | grep ^DOCKER_PASSWORD= | cut -d= -f2- | docker login --password-stdin --username ${DOCKER_USERNAME}"
          sh "/env.sh goreleaser --rm-dist"
        }
      }
    }

    stage('Test Docker image') {
      withEnv(["DOCKER_CONFIG=/tmp/${env.BUILD_TAG}"]) {
        sh "/env.sh docker run --rm defn/hello:${env.GORELEASER_CURRENT_TAG}-amd64"
      }
    }

  }
}

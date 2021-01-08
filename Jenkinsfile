#!/usr/bin/env groovy

import hudson.util.Secret
import com.cloudbees.plugins.credentials.CredentialsScope
import com.datapipe.jenkins.vault.credentials.VaultAppRoleCredential

def NM_ROLE = 'pipeline'
def pipelineRoleId = 'b45fcd66-6e60-3c2f-57e9-c0c5ecd59df2'

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
      def PIPELINE_SECRET_ID= ''
      env.PIPELINE_SECRET_ID = sh(returnStdout: true, script: "./ci/build ${NM_ROLE}").trim()

      VaultAppRoleCredential pipelineCredential = new VaultAppRoleCredential(
        CredentialsScope.GLOBAL,
        'defn--hello-vault', 'defn--hello-vault',
        pipelineRoleId, 
        Secret.fromString(env.PIPELINE_SECRET_ID),
        "approle"
      )

      def pipelineConfiguration = [
        vaultCredential: pipelineCredential
      ]

      withVault([vaultSecrets: pipelineSecrets, configuration: pipelineConfiguration]) {
        sh "env | grep MEH"
      }
    }

    stage ('Prep') {
      sh "install -d -m 0700 /tmp/docker"
      sh "install -d -m 0700 /tmp/docker/${env.BUILD_TAG}"
      sh "git tag ${env.GORELEASER_CURRENT_TAG}"
    }

    withVault([vaultSecrets: githubSecrets]) {
      withEnv(["DOCKER_CONFIG=/tmp/docker/${env.BUILD_TAG}"]) {
        stage('Build') {
          sh "env | grep ^DOCKER_PASSWORD= | cut -d= -f2- | docker login --password-stdin --username ${DOCKER_USERNAME}"
          sh "/env.sh goreleaser build --rm-dist"
        }

        stage('Tests') {
          sh "/env.sh docker run --rm defn/hello:${env.GORELEASER_CURRENT_TAG}-amd64"
        }

        if (env.BRANCH_NAME == 'main') {
          stage('Release') {
            sh "env | grep ^DOCKER_PASSWORD= | cut -d= -f2- | docker login --password-stdin --username ${DOCKER_USERNAME}"
            sh "/env.sh goreleaser release"
          }
        }
        else {
          stage('Build Docker image') {
            sh "docker build -t defn/hello:${env.GORELEASER_CURRENT_TAG}-amd64 ."
          }
        }

        stage('Test Docker image') {
          sh "/env.sh docker run --rm defn/hello:${env.GORELEASER_CURRENT_TAG}-amd64"
        }
      }
    }

    stage('Cleanup') {
      sh "rm -rf /tmp/docker/${env.BUILD_TAG}"
    }
  }
}

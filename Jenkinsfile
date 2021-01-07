#!/usr/bin/env groovy

def NM_ROLE = 'pipeline'
def ID_ROLE = 'b45fcd66-6e60-3c2f-57e9-c0c5ecd59df2'

def secrets = [
  [ 
    path: 'kv/defn/hello',
    secretValues: [
      [envVar: 'MEH', vaultKey: 'name']
    ]
  ]
]

def vaultConfig = [
  vaultCredentialId: 'VaultToken',
  vaultUrl: env.VAULT_ADDR
]

node() {
  checkout scm

  withCredentials([[
    $class: 'VaultTokenCredentialBinding',
    credentialsId: 'VaultToken',
    vaultAddr: env.VAULT_ADDR ]]) {

    stage ('Secrets') {
      withVault([vaultSecrets: secrets, configuration: vaultConfig]) {
        sh "env | grep NAME"
        sh "echo ${NAME}"
      }

      sh """
        ./ci/build "${NM_ROLE}" "${ID_ROLE}"
      """
    }

    stage('Goreleaser') {
      sh "/env.sh goreleaser --snapshot --rm-dist"
    }

    stage('Docker image') {
      sh "/env.sh docker run --rm defn/hello:${BUILD_TAG}-amd64"
    }

    // inside this block your credentials will be available as env variables
    withVault([configuration: configuration, vaultSecrets: secrets]) {
        sh 'echo $testing'
        sh 'echo $testing_again'
        sh 'echo $another_test'
    }
  }
}

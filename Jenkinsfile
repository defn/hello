#!/usr/bin/env groovy

node() {
  withCredentials([[
      $class: 'VaultTokenCredentialBinding',
      credentialsId: 'VaultToken'
      vaultAddr: 'http://127.0.0.1:8200'
    ]]) {

    stage ('Vault Token Lookup') {
      sh(
        returnStdout: true,
        script: "vault token lookup"
      )
    }
  }          
}

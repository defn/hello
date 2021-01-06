#!/usr/bin/env groovy

def ROLE = 'pipeline'
def ROLE_ID = 'b45fcd66-6e60-3c2f-57e9-c0c5ecd59df2'

node() {
  withCredentials([[
      $class: 'VaultTokenCredentialBinding',
      credentialsId: 'VaultToken',
      vaultAddr: env.VAULT_ADDR ]]) {

    stage ('Read Secrets') {
      sh 'vault token lookup'
    }

    stage ('Pipeline wrapped secret_id') {
        def WRAPPED_SID = ''
        env.WRAPPED_SID = sh(
          returnStdout: true,
          script: "vault write -field=wrapping_token -wrap-ttl=200s -f auth/approle/role/${ROLE}/secret-id"
        )
    }

    stage('Pipeline unwrap secret_id') {
      def UNWRAPPED_SID = ''
      env.UNWRAPPED_SID = sh(
        returnStdout: true,
        script: "vault unwrap -field=secret_id ${WRAPPED_SID}"
      )
    }

    stage('Pipeline approle login token') {
      def VAULT_LOGIN_TOKEN = ''
      env.VAULT_LOGIN_TOKEN = sh(
        returnStdout: true,
        script: "vault write -field=token auth/approle/login role_id=${ROLE_ID} secret_id=${UNWRAPPED_SID}"
      )
    }

    stage('Pipeline approle login') {
      def VAULT_PIPELINE_TOKEN = ''
      env.VAULT_PIPELINE_TOKEN = sh(
        returnStdout: true,
        script: "vault login -field=token ${VAULT_LOGIN_TOKEN}"
      )
    }

    stage ('Pipeline token lookup') {
      sh "env VAULT_TOKEN=${VAULT_PIPELINE_TOKEN} vault token lookup"
    }

  }
}

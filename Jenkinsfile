@Library('jenkins-shared-lib') _

pipeline {
  agent {
    label 'docker-agent-client'
  }

  parameters {
    booleanParam(name: 'DRY_RUN', defaultValue: false, description: 'Simulación (no copia)')
    booleanParam(name: 'MIRROR',  defaultValue: false, description: 'Modo espejo (borra extras en destino)')
    string(name: 'HA_HOST', defaultValue: '192.168.1.150', description: 'Host de Home Assistant')
    string(name: 'HA_SSH_PORT', defaultValue: '2222', description: 'Puerto SSH del add-on Terminal & SSH')
    string(name: 'HA_CRED_ID', defaultValue: 'ha_jenkins', description: 'ID de la credencial SSH en Jenkins')
  }

  options {
    timestamps()
  }

  environment {
    // Notificaciones (Telegram u otro backend definido en shared-lib)
    TOKEN = credentials('telegram-chat-token')
    CHAT_ID = credentials('telegram-chat-id')

    // Destino SSH en Home Assistant (parametrizado)
    HA_HOST = "${params.HA_HOST}"
    HA_SSH_PORT = "${params.HA_SSH_PORT}"
    DST_DIR = '/config/custom_templates'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Validar origen') {
      steps {
        sh '''#!/usr/bin/env bash
set -euo pipefail

echo "Ficheros a desplegar (.jinja):"
ls -1 "${WORKSPACE}"/*.jinja 2>/dev/null || { echo "No se encontraron .jinja"; exit 1; }
'''
      }
    }

    stage('Desplegar a HA (SSH/rsync)') {
      steps {
        script {
          haCopy.rsync(
            host: env.HA_HOST,
            port: env.HA_SSH_PORT,
            userCredentialsId: params.HA_CRED_ID,
            srcDir: env.WORKSPACE,
            dstDir: env.DST_DIR,
            dryRun: params.DRY_RUN,
            mirror: params.MIRROR,
            include: ['*.jinja'],
            exclude: ['*']
          )
        }
      }
    }
  }

  post {
    success { script { notify.success() } }
    failure { script { notify.failure() } }
  }
}

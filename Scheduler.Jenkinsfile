// Job orquestrador/despachador/agendador/escalonador/nome a definir
pipeline {
  agent any

  triggers {
    cron('55 3 * * *')
  }

  stages {
    stage('Despachar') {
      steps {
        script {
          // Se for Multibranch Pipeline, algo como:
          // build job: 'Eberick/trunk',
          // build job: 'Eberick/2025-11',
          build job: 'Eberick',
          wait: false,
          parameters: [
            string(name: "TIPO_BUILD", value: "DEBUG")
          ]
        }
      }
    }
  }
}

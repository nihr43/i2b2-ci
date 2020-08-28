properties([
  // https://github.com/jenkinsci/pipeline-model-definition-plugin/wiki/Parametrized-pipelines
  parameters([
    booleanParam(name: 'reprovision_database', defaultValue: true)
  ])
])

pipeline {
  agent any
  stages {
    stage('setup terraform') {
      // terraform statefiles are picky about version, so we pin it in the Makefile
      steps {
        sh 'make terraform'
        sh 'make .terraform'
      }
    }
    stage('terraform plan') {
      environment {
        TF_VAR_PSQL_PASS    = credentials('TF_VAR_PSQL_PASS')
        TF_VAR_I2B2_DB_PASS = credentials('TF_VAR_I2B2_DB_PASS')
      }
      steps {
        sh 'if [ "$reprovision_database" == "true" ] ; then ./terraform taint postgresql_database.i2b2 ; fi'
        sh 'make plan'
      }
    }
    stage('terraform apply') {
      input {
        message "accept plan?"
        ok "yes"
      }
      environment {
        TF_VAR_PSQL_PASS    = credentials('TF_VAR_PSQL_PASS')
        TF_VAR_I2B2_DB_PASS = credentials('TF_VAR_I2B2_DB_PASS')
      }
      steps {
        sh 'make force-apply'
	sh 'make force-apply' // hack: if a db gets re-provisioned, the schemas do not until next run
      }
    }
    stage('load i2b2-data') {
      when {
        environment name: 'reprovision_database', value: 'true'
      }
      environment {
        TF_VAR_I2B2_DB_PASS = credentials('TF_VAR_I2B2_DB_PASS')
      }
      steps {
        sh 'make i2b2'
      }
    }
  }
}

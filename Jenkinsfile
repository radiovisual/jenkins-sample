import org.jenkinsci.plugins.pipeline.modeldefinition.Utils

def dockerImage = "nexus.example.com:5000/web/auth"
def standardEnvironments = ['staging', 'preproduction', 'production'];
def defaultConfig = [lint: true, test: true, build: true, deploy: false, chart: 'web-auth', charts_repository: 'https://example.com']

node ('jenkins') {
  properties([
    parameters([
      booleanParam(name: 'lint', defaultValue: false, description: 'Execute linting stage'),
      booleanParam(name: 'test', defaultValue: false, description: 'Execute test stage'),
      booleanParam(name: 'build', defaultValue: false, description: 'Execute build stage'),
      booleanParam(name: 'deploy', defaultValue: false, description: 'Execute deploy stage'),
      [$class: 'GitParameterDefinition', name: 'deploy_tag', defaultValue: '', selectedValue: 'NONE', sortMode: 'DESCENDING_SMART', description: 'Which GitHub Release Tag to deploy', type: 'PT_TAG', quickFilterEnabled: true],
      choiceParam(name: 'target', choices: (['choose:', 'sandbox', 'custom namespace'] + standardEnvironments).join('\n'), defaultValue: 'choose:', description: 'Which environment to deploy to'),
      stringParam(name: 'host', defaultValue: '', description: '$host.kubernetes.example.com'),
      stringParam(name: 'custom_namespace', defaultValue: '', description: 'Custom target namespace name\n💡 Only required when target "custom namespace" is selected'),
      stringParam(name: 'custom_values_override', defaultValue: '', description: 'Custom chart values override filename\n💡 Only required when target "custom namespace" is selected'),
      stringParam(name: 'chart', defaultValue: defaultConfig.chart, description: 'Helm Chart name'),
      stringParam(name: 'charts_repository', defaultValue: defaultConfig.charts_repository, description: 'Helm Charts repository URL'),
      booleanParam(name: 'confirm', defaultValue: false, description: 'Confirm manual run\n🚨 Please review you settings carefully!')
    ])
  ])

  stage('checkout') {
    sh "echo checkout stage"

    echo """
    =====================================================                                                
    lint       | ${!config.lint ? '❌' : '✅' }
    test       | ${!config.test ? '❌' : '✅' }
    build      | ${!config.build ? '❌' : '✅' }
    tag        | ${!config.tag ? '❌' : config.tag}
    deploy     | ${!config.deploy ? '❌' : '✅'}
    -----------------------------------------------------
    target     : ${!config.deploy ? '' : config.target}
    deploy_tag : ${!config.deploy ? '' : config.deploy_tag}
    hash       : ${config.hash}
    =====================================================
    """
  }

  stage ('prepare') {
    if (!(config.lint || config.test)) return skipStage()

    sh "echo prepare stage"
  }
  parallel (
    lint: {
      stage ('lint') {
        sh "echo lint stage"
      }
    },
    test: {
      stage ('test') {
        if (!config.test) return skipStage()

        sh "echo test stage"
      }
    },
    build: {
      stage ('build') {
        if (!config.build) return skipStage()
        
        sh "echo build stage"
      }
    },
  )

  stage ('reports') {
    if (!(config.lint || config.test)) return skipStage()

    sh "echo reports stage"
  }

  stage ('tag') {
    if (!config.tag) return skipStage()

    sh "echo tag stage"
  }

  stage ('deploy') {
    if (!config.deploy) return skipStage();

    sh "echo deploy stage"
    
  }
}

def skipStage() {
  return Utils.markStageSkippedForConditional(STAGE_NAME)
}

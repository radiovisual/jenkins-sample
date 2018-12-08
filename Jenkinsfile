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
    
    config = defaultConfig + [branch: env.BRANCH_NAME, tag: env.TAG_NAME, force: true]

    config.hash = checkout(scm).GIT_COMMIT
    config.shortHash = sh(returnStdout: true, script: "git rev-parse --short ${config.hash}").trim()

    if (params.confirm) { // manually parametrized job
      config << params
      config << [
        deploy_tag: params.deploy_tag ? normalize(text: params.deploy_tag, dot: false, cut: false) : config.shortHash,
        host: config.host == '' ? normalize(text: config.branch) : params.host,
        target: config.target == 'custom namespace' ? params.custom_namespace : config.target,
        custom_namespace_target: true
      ]

    } else if (config.tag) { // automatic run for git tag 
      config.tag = normalize(text: config.tag, dot: false, cut: false)
      config << [lint: false, test: false, build: false]
      config << [deploy: true, deploy_tag: config.tag, target: 'preproduction']

    } else if (config.branch == 'master') { // automatic run for master
      config.tag = normalize(text: 'latest', dot: false, cut: false)
      config << [deploy: true, deploy_tag: config.shortHash, target: 'staging']

    } else if (config.branch.toLowerCase().startsWith('release')) { // automatic run for release branches
      config << [deploy: true, deploy_tag: config.shortHash, target: 'preproduction']
    }

    unstable = false
    env.GIT_COMMIT = config.hash
    

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

def normalize(Map args) {
  def options = [text: '', slash: true, dot: true, cut: true, lower: true] + args
  
  result = "${options.text}"
  
  if (options.slash) result = result.replaceAll('/', '-')
  if (options.dot) result = result.replaceAll('\\.', '-')
  if (options.cut) result = sh (script: "echo \"${result}\" | cut -d '-' -f1,2,3", returnStdout: true).trim()
  if (options.lower) result = result.toLowerCase()
  
  return result
}

#!/usr/bin/env groovy

node {

  def artServer = Artifactory.newServer url: SERVER_URL, credentialsId: CREDENTIALS
  def buildInfo = Artifactory.newBuildInfo()

  stage ('Cleanup')
  {
      cleanWs()
  }

  stage ('Clone')
  {
        git url: 'https://github.com/jfrog/SolEngDemo.git',
        branch: 'master',
        credentialsId: GIT_CREDENTIALS
  }

  stage ('Build Gradle')
  {
    dir ('./Samples/step1-create-gradle-app/'){
      def artifactoryGradle = Artifactory.newGradleBuild()
      artifactoryGradle.tool = 'GRADLE_TOOL' // Tool name from Jenkins configuration
      artifactoryGradle.deployer repo:'gradle-dev-local-decl', server: artServer
      artifactoryGradle.resolver repo:'gradle-release-decl', server: artServer
      artifactoryGradle.run rootDir: ".", buildFile: 'build.gradle', tasks: 'clean build artifactoryPublish --stacktrace', buildInfo: buildInfo

      

      buildInfo.env.capture = true
     
      buildInfo.env.collect()

      artServer.publishBuildInfo buildInfo
    }
  }

  stage ('Xray scan')
  {
        if (XRAY_SCAN == "YES") {
           def xrayConfig = [
              'buildName'     : env.JOB_NAME,
              'buildNumber'   : env.BUILD_NUMBER,
              'failBuild'     : false
            ]
            def xrayResults = artServer.xrayScan xrayConfig
            echo xrayResults as String
            sleep 10
       } else {
            println "No Xray scan performed. To enable set XRAY_SCAN = YES"
       }
  }

  stage ('Test')
  {
    // testing here

  }

  stage ('Promotion')
  {
    def promotionConfig = [
        //Mandatory parameters
        'buildName'          : buildInfo.name,
        'buildNumber'        : buildInfo.number,

        //Optional parameters
        'targetRepo'         : 'gradle-release-local-decl',
        'comment'            : 'Promoting Gradle Build',
        'sourceRepo'         : 'gradle-dev-local-decl',
        'status'             : 'Released',
        'includeDependencies': false,
        'failFast'           : true,
        'copy'               : true
    ]

    // Promote Gradle Build
    artServer.promote promotionConfig
  }
  stage ('Report back to JFrog Pipelines')
  {
    jfPipelines(
      outputResources: """[
       {
         "name": "gradle_jenkins_fis_results",
         "content": {
           "passing": 17,
           "failing": 0
         }
       }
      ]"""
   )
  }
}

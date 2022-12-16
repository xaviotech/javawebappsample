import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=a5cdc8eb-472c-4b8b-a3a8-70c2ba30f7bb',
        'AZURE_TENANT_ID=8a09f2d7-8415-4296-92b2-80bb4666c5fc']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      bat 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'test-rg'
      def webAppName = 'xavi-0ne'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '1a3c677d-fc21-445c-b399-910c13b9c221', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
       bat '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = bat script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      bat "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      bat 'az logout'
    }
  }
}

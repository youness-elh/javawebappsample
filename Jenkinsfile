import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=9a86476e-b022-4aa8-9372-dab324cf625d',
        'AZURE_TENANT_ID=dab7d5be-3d32-41f6-b6b4-f31ecf90c7af']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg-y'
      def webAppName = 'appyouness'
      // login Azure
      /*
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'd2v8Q~gpVsK4-.FKhSgGIjXXlmLkqKb4Ijo7gcnV', usernameVariable: '7454a61d-a99c-45d0-82e1-321eea3031f4')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      */
      withCredentials([azureServicePrincipal(credentialsId: 'AzureServicePrincipal', variable: 'AZURE_CREDENTIALS')]) {
      sh '''
        az login --service-principal -u $AZURE_CREDENTIALS_CLIENT_ID -p $AZURE_CREDENTIALS_CLIENT_SECRET -t $AZURE_CREDENTIALS_TENANT_ID
        az account set -s $AZURE_CREDENTIALS_SUBSCRIPTION_ID
      '''
}

      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}

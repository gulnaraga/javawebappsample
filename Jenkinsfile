import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=751fab54-447d-4b8f-8d44-12172466e856',
        'AZURE_TENANT_ID=a97efdaf-6e17-4162-bede-340fb893e8a8']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'myjava-jenksin-app09'
      // login Azure
       withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'Pl1e62jka8ktL~eh8a5Yp3R8on52hEvFHc', usernameVariable: '521d8cc2-6e86-470c-95ca-ead52a9124bd')]) {
       sh '''
          az login --service-principal -u 521d8cc2-6e86-470c-95ca-ead52a9124bd -p Pl1e62jka8ktL~eh8a5Yp3R8on52hEvFHc -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
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

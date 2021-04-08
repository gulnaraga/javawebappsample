import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=82871f4a-6f7f-44df-a6e6-4d21f47bd2a2',
        'AZURE_TENANT_ID=96685ea3-b366-4de3-8a70-cd02922d6b52']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'QuickstartJenkins-rg'
      def webAppName = 'newjenkins-app-ey8'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'JHm12luIPY-bWtR7Q5FaAPM-kDytiPozxr', usernameVariable: 'db8c1084-07a0-4d2c-9e84-3ada7ed688c5')]) {
       sh '''
          az login --service-principal -u db8c1084-07a0-4d2c-9e84-3ada7ed688c5 -p JHm12luIPY-bWtR7Q5FaAPM-kDytiPozxr -t 96685ea3-b366-4de3-8a70-cd02922d6b52
          az account set -s 82871f4a-6f7f-44df-a6e6-4d21f47bd2a2
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

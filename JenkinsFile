node {
    
    def project_path = 'spring-boot-samples/spring-boot-sample-atmosphere'
    def forked_github_repo = 'https://github.com/sreevesds9/jenkins2-course-spring-boot'
    
 try {
     
    currentBuild.result = 'Success'
    
    stage("checkout from github") {
     git "$forked_github_repo"
    }
    
    dir("$project_path") {
        
       stage("compile") {
        sh 'mvn compile'
       }
       
       stage("build") {
        sh 'mvn clean package'
       }
    
    
       stage("Archive jar file") {
    
       step([$class: 'ArtifactArchiver',
                   artifacts: "target/*.jar",
                   excludes: null]);
       }
       
       
       timeout(time:5, unit:'MINUTES') {
        input message:'Approve deployment?', submitter: 'it-ops'
       }
       
       stage("Publish to AWS S3 bucket") {
           
        step([$class: 'S3BucketPublisher',
        entries: [[sourceFile: 'target/*.jar',
        bucket: 'jenkins2',
        selectedRegion: 'us-west-2',
        noUploadOnFailure: true,
        managedArtifacts: false,
        flatten: true,
        showDirectlyInBrowser: true,
        keepForever: true]],
        profileName: 'jenkins2',
        dontWaitForConcurrentBuildCompletion: false, 
        ])
       }

    }
 } catch (err) {
     stage("Email Notification")
     notify("ERROR: ${err}")
     currentBuild.result = "Failure"
     throw err

 }
     stage("Email Notification") {
      notify('Success')
     }
   
    
}

def notify(status) {
    
    emailext (
      to: "sreeves@epnet.com",
      subject: "${status} ${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status} Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
    
}

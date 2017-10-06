node('master') {
    stage('git checkout') {
        checkout scm
    }
    stage('build dacpac') {
        bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
        stash includes: 'SelfBuildPipelineDV\\bin\\Release\\SelfBuildPipelineDV.dacpac', name: 'theDacpac'
    }
}

node ('linux-slave') {
    stage('Start Container') {
        sh 'docker volume create --driver=pure -o size=4GB datavol'
        sh 'docker run -v datavol:/data -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=P@ssword1" --name mssqllinux -d -i -p 15565:1433 microsoft/mssql-server-linux:2017-latest'
    }
}

node('master') {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "SelfBuildPipelineDV\\bin\\Release\\SelfBuildPipelineDV.dacpac"
    def ConnString = "server=linux-slave,15565;database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
 
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}

node ('linux-slave') {
    stage('Clean up') {
        sh 'docker rm -f mssqllinux'
        sh 'docker volume rm datavol'
    }
}

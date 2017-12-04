def PowerShell(psCmd) {
    bat "powershell.exe -NonInteractive -ExecutionPolicy Bypass -Command \"\$ErrorActionPreference='Stop';$psCmd;EXIT \$global:LastExitCode\""
}

def BranchToPort(String branchName) {
    def BranchPortMap = [
        [branch: 'master'   , port: 15565],
        [branch: 'Release'  , port: 15566],
        [branch: 'Feature'  , port: 15567],
        [branch: 'Prototype', port: 15568],
        [branch: 'HotFix'   , port: 15569]
    ]
    BranchPortMap.find { it['branch'] ==  branchName }['port']
}
 
def StartContainer() {
    PowerShell "If (\$((docker ps -a --filter \"name=SQLLinux${env.BRANCH_NAME}\").Length) -eq 2) { docker rm -f SQLLinux${env.BRANCH_NAME} }"
    sh "docker volume create --driver=pure -o size=4GB ${env.BRANCH_NAME}"
    docker.image('microsoft/mssql-server-linux:2017-GA').run("-v ${env.BRANCH_NAME}:/var/opt/mssql -e ACCEPT_EULA=Y -e SA_PASSWORD=P@ssword1 --name SQLLinux${env.BRANCH_NAME} -d -i -p  ${BranchToPort(env.BRANCH_NAME)}:1433")
    PowerShell "While (\$((docker logs SQLLinux${env.BRANCH_NAME} | select-string ready | select-string client).Length) -eq 0) { Start-Sleep -s 1 }"
}
 
def DeployDacpac() {
    def SqlPackage = "C:\\Program Files\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe"
    def SourceFile = "SelfBuildPipelineDV\\bin\\Release\\SelfBuildPipelineDV.dacpac"
    def ConnString = "server=linux-slave,${BranchToPort(env.BRANCH_NAME)};database=SsdtDevOpsDemo;user id=sa;password=P@ssword1"
 
    unstash 'theDacpac'
    bat "\"${SqlPackage}\" /Action:Publish /SourceFile:\"${SourceFile}\" /TargetConnectionString:\"${ConnString}\" /p:ExcludeObjectType=Logins"
}
 
node {
    stage('git checkout') {
        timeout(time: 5, unit: 'SECONDS') {
            checkout scm
        }
    }
    stage('build dacpac') {
        bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
        stash includes: 'SelfBuildPipelineDV\\bin\\Release\\SelfBuildPipelineDV.dacpac', name: 'theDacpac'
    }
}

node('linux-slave') {
    stage('start container') {
        timeout(time: 20, unit: 'SECONDS') {
            StartContainer()
        }
    }
}

node {
    stage('Deploy DACPAC') {
        timeout(time: 60, unit: 'SECONDS') {
            DeployDacpac()
        }
    }
}

node ('linux-slave') {
    stage('Clean up') {
        sh "docker rm -f SQLLinux${env.BRANCH_NAME}"
    }
}

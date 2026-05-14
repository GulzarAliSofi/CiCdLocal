pipeline {
    agent any
    environment {
        DOTNET_CLI_HOME = "${env.WORKSPACE}"
        DOTNET_NOLOGO = 'true'
    }
    stages {
        stage('Restore') {
            steps {
                bat 'dotnet --version'
                bat 'dotnet restore %WORKSPACE%\\CiCdLocal.sln'
            }
        }
        stage('Build') {
            steps {
                bat 'dotnet build %WORKSPACE%\\CiCdLocal.sln --no-restore --configuration Release'
            }
        }
        stage('Test') {
            steps {
                bat 'dotnet test %WORKSPACE%\\tests\\SimpleWebApi.Test\\SimpleWebApi.Test.csproj --no-build --no-restore --configuration Release --collect "XPlat Code Coverage" --logger "trx;LogFileName=test-results.trx"'
            }
            post {
                always {
                    // Archive test results and coverage reports
                    archiveArtifacts artifacts: 'tests/SimpleWebApi.Test/TestResults/**/*.trx', allowEmptyArchive: true
                    archiveArtifacts artifacts: 'tests/SimpleWebApi.Test/TestResults/**/coverage.cobertura.xml', allowEmptyArchive: true
                }
            }
        }
        stage('Deliver') {
            steps {
                bat 'dotnet publish %WORKSPACE%\\src\\CiCdLocal\\CiCdLocal.csproj --no-build --no-restore --configuration Release -o published'
                echo "Validating published artifacts..."
                bat 'dir %WORKSPACE%\\published'
                echo "Verifying main executable exists..."
                bat 'if exist %WORKSPACE%\\published\\CiCdLocal.exe (echo API executable found!) else (exit /b 1)'
            }
            post{
                success {
                    archiveArtifacts artifacts: 'src/CiCdLocal/published/**/*', allowEmptyArchive: true
                }
            }
        }
    }
    post {
        always {
            bat 'dotnet clean %WORKSPACE%\\CiCdLocal.sln'
        }
    }
}
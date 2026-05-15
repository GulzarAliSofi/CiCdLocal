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
         stage('Deploy to IIS') {
            steps {
                bat '''
                echo Stopping IIS...
                iisreset /stop

                net stop w3svc

                echo Creating destination folder if it does not exist...
                if not exist "C:\\inetpub\\wwwroot\\CiCdWebApi" mkdir "C:\\inetpub\\wwwroot\\CiCdWebApi"

                echo Copying published files to wwwroot using PowerShell...
                powershell -NoProfile -Command "try { Copy-Item -Path '%WORKSPACE%\\published\\*' -Destination 'C:\\inetpub\\wwwroot\\CiCdWebApi' -Force -Recurse; Write-Host 'Files copied successfully'; (Get-ChildItem -Path 'C:\\inetpub\\wwwroot\\CiCdWebApi' -Force | Measure-Object).Count | ForEach-Object { Write-Host 'Total files deployed:' $_ } } catch { Write-Host 'Copy failed:' $_; exit 1 }"

                net start w3svc
                
                echo Starting IIS...
                iisreset /start

                echo Validating Web API endpoint...
                powershell -NoProfile -Command "try { $r = Invoke-WebRequest -UseBasicParsing -Uri 'http://localhost:809/api/WeatherForecast' -TimeoutSec 15; if ($r.StatusCode -eq 200) { Write-Host 'API validation passed with status' $r.StatusCode; exit 0 } else { throw 'Unexpected status code: ' + $r.StatusCode } } catch { Write-Host 'API validation failed:' $_; exit 1 }"
                '''
            }
        }
    }
    post {
        always {
            bat 'dotnet clean %WORKSPACE%\\CiCdLocal.sln'
        }
    }
}
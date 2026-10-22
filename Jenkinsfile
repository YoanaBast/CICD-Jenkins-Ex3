pipeline {
    agent any

    environment {
        CHROME_VERSION = '126.0.6478.126'
        DOTNET_ROOT    = "${HOME}/.dotnet"
        PATH           = "${HOME}/.dotnet:${PATH}"
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set up .NET') {
            steps {
                sh '''
                    curl -sSL https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh
                    chmod +x dotnet-install.sh
                    ./dotnet-install.sh --channel 6.0
                    dotnet --version
                '''
            }
        }

        stage('Uninstall current Chrome') {
            steps {
                sh '''
                    sudo apt-get remove -y google-chrome-stable || true
                    sudo rm -rf /opt/chrome /opt/chromedriver
                    sudo rm -f /usr/local/bin/google-chrome /usr/local/bin/chromedriver
                '''
            }
        }

        stage('Install specific Chrome') {
            steps {
                sh '''
                    sudo apt-get update
                    sudo apt-get install -y unzip curl
                    curl -sSL -o chrome.zip https://storage.googleapis.com/chrome-for-testing-public/${CHROME_VERSION}/linux64/chrome-linux64.zip
                    sudo unzip -q -o chrome.zip -d /opt
                    sudo mv /opt/chrome-linux64 /opt/chrome
                    sudo ln -sf /opt/chrome/chrome /usr/local/bin/google-chrome
                    google-chrome --version
                '''
            }
        }

        stage('Install ChromeDriver') {
            steps {
                sh '''
                    curl -sSL -o chromedriver.zip https://storage.googleapis.com/chrome-for-testing-public/${CHROME_VERSION}/linux64/chromedriver-linux64.zip
                    sudo unzip -q -o chromedriver.zip -d /opt
                    sudo mv /opt/chromedriver-linux64 /opt/chromedriver
                    sudo ln -sf /opt/chromedriver/chromedriver /usr/local/bin/chromedriver
                    chromedriver --version
                '''
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --no-restore --configuration Release'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test --no-build --configuration Release --logger "trx;LogFileName=results.trx"'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/TestResults/*.trx', allowEmptyArchive: true
        }
    }
}
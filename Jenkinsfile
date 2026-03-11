pipeline {
    agent { label 'build-slave-01' } // THAY THẾ: Tên nhãn Agent của bạn

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Branch để deploy')
    }

    environment {
        // Cấu hình đường dẫn thực tế trên Server Agent
        BUILD_USER      = "msbuild"
        SERVICE_USER    = "msservice"
        DEPLOY_BASE_DIR = "/home/msservice/my-app"
    }

    stages {
        stage('1. Prepare & Identity') {
            steps {
                script {
                    echo "--- Running on Node: ${env.NODE_NAME}"
                    echo "--- User Jenkins: ${sh(script: 'whoami', returnStdout: true).trim()}"
                }
            }
        }

        stage('2. Build Stage (Acting as msbuild)') {
            steps {
                echo "================================================="
                echo "*** Simulation Compile/Build code ***"
                echo "================================================="

                sh """
                        # 1. change owner
                        sudo chown -R ${BUILD_USER}:${BUILD_USER} ${WORKSPACE}
                        
                        # 2. write with msbuild
                        sudo -u ${BUILD_USER} bash -c "echo 'Build version 1.0.0' > build_report.txt"
                        
                        # 3. test
                        sudo -u ${BUILD_USER} ls -la ${WORKSPACE}
                    """
            }
        }

        stage('3. Deploy Stage (Acting as msservice)') {
            steps {
                echo "================================================="
                echo "*** Bước này đẩy code sang thư mục Production ***"
                echo "================================================="
                sh """
                    # Tạo thư mục đích nếu chưa có
                    sudo mkdir -p ${DEPLOY_BASE_DIR}
                    sudo chown ${SERVICE_USER}:${SERVICE_USER} ${DEPLOY_BASE_DIR}

                    # Dùng rsync để đồng bộ code sang thư mục chạy App
                    # Lệnh chạy dưới quyền msservice để đảm bảo msservice sở hữu file mới
                    sudo -u ${SERVICE_USER} rsync -avz --delete ${WORKSPACE}/ ${DEPLOY_BASE_DIR}/ \
                        --exclude .git \
                        --exclude Jenkinsfile

                    echo "--- Kiểm tra thư mục Deploy sau khi hoàn tất:"
                    sudo -u ${SERVICE_USER} ls -l ${DEPLOY_BASE_DIR}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Chúc mừng! Code đã được deploy bởi user ${SERVICE_USER} thành công."
        }
        failure {
            echo "❌ Có lỗi xảy ra. Kiểm tra lại quyền sudo hoặc đường dẫn thư mục."
        }
    }
}

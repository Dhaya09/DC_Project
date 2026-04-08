pipeline {
    agent any
    stages {
        stage('Start Cluster') {
            steps {
                bat 'docker start namenode datanode1 datanode2 datanode3'
                bat 'docker exec datanode1 service ssh start'
                bat 'docker exec datanode2 service ssh start'
                bat 'docker exec datanode3 service ssh start'
                bat 'docker exec namenode service ssh start'
                bat 'docker exec namenode /opt/hadoop/sbin/start-dfs.sh'
                bat 'docker exec namenode /opt/hadoop/sbin/start-yarn.sh'
            }
        }
        stage('Prepare HDFS') {
            steps {
                bat 'ping -n 15 127.0.0.1 > nul'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfsadmin -safemode leave'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -rm -r -f /user/student/output'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -mkdir -p /user/student/input'
                bat 'docker cp C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\student_dataset.csv namenode:/opt/results/student_dataset.csv'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -put -f /opt/results/student_dataset.csv /user/student/input/'
            }
        }
        stage('Run MapReduce') {
            steps {
                bat 'docker exec namenode /opt/hadoop/bin/hadoop jar /opt/mapreduce/AcademicPerformance.jar AcademicPerformance /user/student/input /user/student/output/academic'
                bat 'docker exec namenode /opt/hadoop/bin/hadoop jar /opt/mapreduce/RiskFactorAnalysis.jar RiskFactorAnalysis /user/student/input /user/student/output/risk'
            }
        }
        stage('Generate Reports') {
            steps {
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/student/output/academic/part-r-00000 /opt/results/academic_part0.txt'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/student/output/academic/part-r-00001 /opt/results/academic_part1.txt'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/student/output/academic/part-r-00002 /opt/results/academic_part2.txt'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/student/output/risk/part-r-00000 /opt/results/risk_part0.txt'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/student/output/risk/part-r-00001 /opt/results/risk_part1.txt'
                bat 'docker exec namenode /opt/hadoop/bin/hdfs dfs -get -f /user/student/output/risk/part-r-00002 /opt/results/risk_part2.txt'
                bat 'docker exec namenode bash -c "/opt/hadoop/bin/hdfs dfs -cat /user/student/output/academic/part-r-* > /opt/results/academic_final.txt"'
                bat 'docker exec namenode bash -c "/opt/hadoop/bin/hdfs dfs -cat /user/student/output/risk/part-r-* > /opt/results/risk_final.txt"'
                bat 'docker exec namenode python3 /opt/results/generate_csv.py'
                bat 'docker exec namenode python3 /opt/results/generate_report.py'
                bat 'docker exec namenode python3 /opt/results/send_report.py'
            }
        }
        stage('Copy Results to Local') {
            steps {
                bat 'mkdir C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results || exit 0'
                bat 'docker cp namenode:/opt/results/academic_final.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\academic_final.txt'
                bat 'docker cp namenode:/opt/results/risk_final.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\risk_final.txt'
                bat 'docker cp namenode:/opt/results/academic_performance.csv C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\academic_performance.csv'
                bat 'docker cp namenode:/opt/results/risk_factor.csv C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\risk_factor.csv'
                bat 'docker cp namenode:/opt/results/student_report.pdf C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\student_report.pdf'
                bat 'docker cp namenode:/opt/results/academic_part0.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\academic_part0.txt'
                bat 'docker cp namenode:/opt/results/academic_part1.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\academic_part1.txt'
                bat 'docker cp namenode:/opt/results/academic_part2.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\academic_part2.txt'
                bat 'docker cp namenode:/opt/results/risk_part0.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\risk_part0.txt'
                bat 'docker cp namenode:/opt/results/risk_part1.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\risk_part1.txt'
                bat 'docker cp namenode:/opt/results/risk_part2.txt C:\\Users\\Dhayanidhi\\OneDrive\\Desktop\\DC_Dataset\\Results\\risk_part2.txt'
            }
        }
        stage('Stop Cluster') {
            steps {
                bat 'docker exec namenode /opt/hadoop/sbin/stop-yarn.sh || exit 0'
                bat 'docker exec namenode /opt/hadoop/sbin/stop-dfs.sh || exit 0'
                bat 'docker stop namenode datanode1 datanode2 datanode3'
            }
        }
    }
    post {
        always {
            bat 'docker exec namenode /opt/hadoop/sbin/stop-yarn.sh || exit 0'
            bat 'docker exec namenode /opt/hadoop/sbin/stop-dfs.sh || exit 0'
            bat 'docker stop namenode datanode1 datanode2 datanode3 || exit 0'
        }
    }
}

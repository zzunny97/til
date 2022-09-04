# MySQL 소스코드 빌드

**:warning: 본 문서는 ubuntu 18.04를 기준으로 작성되었음. **

### 필요한 패키지 설치
``` bash
$ sudo apt-get install libreadline7
$ sudo apt-get install libaio1 libaio-dev
$ sudo apt-get install build-essential cmake libncurses5 libncurses5-dev bison
```

### 소스코드 다운로드 (Community 버전)
``` bash
$ wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.33.tar.gz
$ tar -xvzf mysql-5.7.33.tar.gz
$ cd mysql-5.7.33
```

### Boost 라이브러리 다운로드 및 빌드
``` bash
$ cmake -DDOWNLOAD_BOOST=ON -DWITH_BOOST=/path/to/download/boost -DCMAKE_INSTALL_PREFIX=/path/to/dir
```
(참고) 만약 Boost 라이브러리가 이미 local에 설치되어 있다면, Path만 잡아주면 된다.
```bash
$ cmake -DWITH_BOOST=/path/to/boost -DCMAKE_INSTALL_PREFIX=/path/to/dir
```
### MySQL 빌드 및 기본 설정
1. 나는 모든 코어를 써서 빌드했지만, 실행환경에 맞춰서 j 뒤에 숫자를 조절해주면 된다.
```bash
$ make -j install
```

2. 초기화
- `--datadir` : MySQL data 디렉토리
- `--basedir` : MySQL 설치 디렉토리
```bash
$ ./bin/mysqld --initialize --user=mysql --datadir=/path/to/datadir --basedir=/path/to/basedir
```

3. root 비밀번호 설정
```bash
$ ./bin/mysqld_safe --skip-grant-tables --datadir=/path/to/datadir

$ ./bin/mysql -uroot

root:mysql> set password = password('yourPassword');
root:mysql> quit;
```

4. bashrc 수정
``` bash
$ vi ~/.bashrc
export PATH=/path/to/basedir/bin:$PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/basedir/lib/

$ source ~/.bashrc
```

5. MySQL 설정파일 작성 및 적용
- 설정파일 작성 (하단은 예시파일)
``` bash
$ vi my.cnf

#
# The MySQL database server configuration file
#
[client]
user    = root
port    = 3306
socket  = /tmp/mysql.sock

[mysql]
prompt  = \u:\d>\_

[mysqld_safe]
socket  = /tmp/mysql.sock

[mysqld]
# Basic settings
default-storage-engine = innodb
pid-file        = /path/to/datadir/mysql.pid
socket          = /tmp/mysql.sock
port            = 3306
datadir         = /path/to/datadir/
log-error       = /path/to/datadir/mysql_error.log

#
# Innodb settings
#
# Page size
innodb_page_size=16KB

# Buffer pool settings
innodb_buffer_pool_size=1G
innodb_buffer_pool_instances=8

# Transaction log settings
innodb_log_file_size=100M
innodb_log_files_in_group=2
innodb_log_buffer_size=32M

# Log group path (iblog0, iblog1)
# If you separate the log device, uncomment and correct the path
#innodb_log_group_home_dir=/path/to/logdir/

# Flush settings (SSD-optimized)
# 0: every 1 seconds, 1: fsync on commits, 2: writes on commits
innodb_flush_log_at_trx_commit=0
innodb_flush_neighbors=0
innodb_flush_method=O_DIRECT
```
- 작성한 설정파일을 적용하여 데몬 재시작
``` bash
$ ./bin/mysqladmin -uroot -pyourPassword shutdown
$ ./bin/mysqld_safe --defaults-file=/path/to/my.cnf
```





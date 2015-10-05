### Linux Configuration Check

#### 1. Check swappiness on all your nodes, then set the recommended value
```
echo 1 > /proc/sys/vm/swappiness
echo "vm.swappiness = 1" >> /etc/sysctl.conf
```
#### 2. Set noatime on DN volumes
```
[root@myuan-fcebootcamp-2 ~]# cat /etc/fstab
LABEL=_/   /         ext4    defaults,noatime        1 1
none       /proc     proc    defaults        0 0
none       /sys      sysfs   defaults        0 0
none       /dev/pts  devpts  gid=5,mode=620  0 0
none       /dev/shm  tmpfs   defaults        0 0
/dev/xvdb	/mnt	auto	defaults,nofail,comment=cloudconfig	0	2

mount -o remount /
mount -o remount /data
cat /proc/mounts
```
#### 3. Set reserve space for root on DN volumes to 0
```
tune2fs -m 0 /dev/xvdb
tune2fs -m 0 /dev/xvdc
```

#### 4. Check the user resource limits for max file descriptors and processes
```
echo hdfs - nofile 32768 >> /etc/security/limits.conf
echo mapred - nofile 32768 >> /etc/security/limits.conf
echo hbase - nofile 32768 >> /etc/security/limits.conf
echo hdfs - nproc 32768 >> /etc/security/limits.conf
echo mapred - nproc 32768 >> /etc/security/limits.conf
echo hbase - nproc 32768 >> /etc/security/limits.conf 
```

#### 5. Test forward and reverse lookups for both file-based and DNS name services
```
host 172.26.16.163
host 172.26.16.253
host 172.26.18.247
host 172.26.16.226
host 172.26.19.229

ping localhost
nslookup myuan-fce-1.vpc.cloudera.com
nslookup myuan-fce-2.vpc.cloudera.com
nslookup myuan-fce-3.vpc.cloudera.com
nslookup myuan-fce-4.vpc.cloudera.com
nslookup myuan-fce-5.vpc.cloudera.com
```

#### 6. Enable nscd
```
chkconfig --list nscd
service --status-all |grep nscd
```

### Install Mysql: http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cm_ig_mysql.html

#### 1. sudo yum -y install mysql-server

#### 2. sudo service mysqld stop

#### 3. rm -rf /var/lib/mysql/ib_logfile*

#### 4. vi /etc/my.cnf
```
[mysqld]
transaction-isolation = READ-COMMITTED
#Disabling symbolic-links is recommended to prevent assorted security risks;
#to do so, uncomment this line:
#symbolic-links = 0

key_buffer = 16M
key_buffer_size = 32M
max_allowed_packet = 32M
thread_stack = 256K
thread_cache_size = 64
query_cache_limit = 8M
query_cache_size = 64M
query_cache_type = 1

max_connections = 550

#log_bin should be on a disk with enough free space. Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your system and chown the specified folder to the mysql user.
#log_bin=/var/lib/mysql/mysql_binary_log
#expire_logs_days = 10
#max_binlog_size = 100M

# For MySQL version 5.1.8 or later. Comment out binlog_format for older versions.
# binlog_format = mixed

read_buffer_size = 2M
read_rnd_buffer_size = 16M
sort_buffer_size = 8M
join_buffer_size = 8M

# InnoDB settings
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit  = 2
innodb_log_buffer_size = 64M
innodb_buffer_pool_size = 4G
innodb_thread_concurrency = 8
innodb_flush_method = O_DIRECT
innodb_log_file_size = 512M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```
#### 5. sudo /sbin/chkconfig mysqld on

#### 6. sudo service mysqld start

#### 7. sudo /usr/bin/mysql_secure_installation
```
[...]
Enter current password for root (enter for none):
OK, successfully used password, moving on...
[...]
Set root password? [Y/n] y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] Y
[...]
Disallow root login remotely? [Y/n] N
[...]
Remove test database and access to it [Y/n] Y
[...]
Reload privilege tables now? [Y/n] Y
All done!
```

#### 7. Install MySQL JDBC Connector
```
wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.35.tar.gz /usr/share/java/mysql-connector-java.jar
```

#### 8. mysql -u root -p
```
create database amon DEFAULT CHARACTER SET utf8;
grant all on amon.* to 'amon'@'%' IDENTIFIED BY 'amon';
create database rman DEFAULT CHARACTER SET utf8;
grant all on rman.* to 'rman'@'%' IDENTIFIED BY 'rman';
create database metastore DEFAULT CHARACTER SET utf8;
grant all on metastore.* to 'hive'@'%' IDENTIFIED BY 'hive';
create database sentry DEFAULT CHARACTER SET utf8;
grant all on sentry.* to 'sentry'@'%' IDENTIFIED BY 'sentry';
create database nav DEFAULT CHARACTER SET utf8;
grant all on nav.* to 'nav'@'%' IDENTIFIED BY 'nav';
create database navms DEFAULT CHARACTER SET utf8;
grant all on navms.* to 'navms'@'%' IDENTIFIED BY 'navms';
create database oozie;
grant all privileges on oozie.* to 'oozie'@'localhost' identified by 'oozie';sud
grant all privileges on oozie.* to 'oozie'@'%' identified by 'oozie';
```

### Cloudera Manager Path B installation

#### 1. sudo yum -y install python27

#### 2. wget http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
   scp cloudera-manager.repo root@myuan-edh-1.vpc.cloudera.com:/etc/yum.repos.d/

#### 3. sudo yum -y install oracle-j2sdk1.7

#### 4. sudo yum -y install cloudera-manager-daemons cloudera-manager-server

#### 5. /usr/share/cmf/schema/scm_prepare_database.sh -u root -p mysql scm scm scm

#### 6. sudo service cloudera-scm-server start

#### 7. Configure Cloudera Manager 

### Testing

1. sudo -u hdfs time hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-examples.jar teragen -Dmapreduce.job.maps=8 -Dmapreduce.map.memory.mb=512 -Dmapreduce.map.java.opts.max.heap=409 -Ddfs.block.size=64m 102400000 /results/teragen

2. sudo -u hdfs time hadoop jar /opt/cloudera/parcels/CDH/jars/hadoop-examples.jar terasort -Dmapreduce.job.maps=8 -Dmapreduce.job.reduces=8 -Dmapreduce.map.memory.mb=512 -Dmapreduce.map.java.opts.max.heap=409 -Dmapreduce.reduce.memory.mb=512 -Dmapreduce.reduce.java.opts.max.heap=409 -Ddfs.block.size=64m /results/teragen /results/terasort 1>terasort.out 2>terasort.err

### Kerberize

1. yum -y install krb5-server krb5-libs krb5-auth-dialog

2. sudo vi /etc/krb5.conf
```
[root@myuan-fcebootcamp-1 ~]# cat /etc/krb5.conf
[libdefaults]
default_realm = CHALLENGE.FCE
dns_lookup_kdc = false
dns_lookup_realm = false
ticket_lifetime = 1d 0h 0m 0s
renew_lifetime = 7d 0h 0m 0s
forwardable = true
default_tgs_enctypes = rc4-hmac
default_tkt_enctypes = rc4-hmac
permitted_enctypes = rc4-hmac
udp_preference_limit = 1
[realms]
CHALLENGE.FCE = {
kdc = myuan-fce-1.vpc.cloudera.com
admin_server = myuan-fce-1.vpc.cloudera.com
}
```
3. sudo vi /var/kerberos/krb5kdc/kdc.conf
```
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 CHALLENGE.FCE = {
  #master_key_type = aes256-cts
  max_life = 1d 0h 0m 0s
  max_renewable_life = 7d 0h 0m 0s
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
``` 
4. sudo /usr/sbin/kdb5_util create -s

5. sudo vi /var/kerberos/krb5kdc/kadm5.acl
```
*/admin@CHALLENGE.FCE	*
```

6. /usr/sbin/kadmin.local -q "addprinc root/admin"

7. sudo service krb5kdc start

8. sudo service kadmin start

9. kinit root/admin@CHALLENGE.FCE

10. klist

11. kadmin -q "addprinc -pw cloudera cloudera-scm/admin@CHALLENGE.FCE"

12. Enable Kerberos Wizard

13. kadmin -q "addprinc -pw cloudera hdfs@CHALLENGE.FCE"

14. kinit hdfs@CHALLENGE.FCE

15. hadoop jar /opt/cloudera/parcels/CDH/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 10 100
```
kadmin.local: modprinc -maxrenewlife 90day krbtgt/CLOUDERA.COM
kadmin.local: modprinc -maxrenewlife 90day +allow_renewable hue/<hostname>@YOUR-REALM.COM
```
#### Sentry - http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/sg_sentry_service_install.html

1. Add Sentry Service to Cluster

2. sudo -u hdfs hadoop fs -chmod -R 771 /user/hive/warehouse

3. sudo -u hdfs hadoop fs -chown -R hive:hive /user/hive/warehouse

4. Disable impersonation for HiveServer2 in the Cloudera Manager Admin Console

5. If you are using MapReduce, enable the Hive user to submit MapReduce jobs.

6. If you are using YARN, enable the Hive user to submit YARN jobs.

7. Enable Sentry Service for Hive

8. Enable the Sentry Service for Hue

8. kinit -k -t hive.keytab hive/myuan-fce-1.vpc.cloudera.com@CHALLENGE.FCE

9. Grant permissions
```
beeline> !connect jdbc:hive2://myuan-fce-1.vpc.cloudera.com:10000/default;principal=hive/myuan-fce-1.vpc.cloudera.com@CHALLENGE.FCE
create role analyst_role;
GRANT ROLE analyst_role TO GROUP myuan;
GRANT ALL ON URI 'hdfs:///user/hive/warehouse/sample_07' to role analyst_role; 
GRANT SELECT ON TABLE default.sample_07 to role analyst_role; 
```

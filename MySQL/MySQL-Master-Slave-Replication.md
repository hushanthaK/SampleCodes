##Prerequisite:
  Install mysql on both nodes ( MASTER & SLAVE) ----> sudo apt update && sudo apt install mysql-server && mysql_secure_installation

  mysql port :3306

  Master IP: 192.168.161.131 
  Slave IP: 192.168.161.130


  +------------------+
  |    MASTER-DB1    |          
  +------------------+

STEP-1: sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
	
	bin-address  = 192.168.161.131 (Master IP)
	server-id    = 1

	#######Add at the end of file#############
	log_bin = /var/log/mysql/mysql-bin.log
	log_bin_index =/var/log/mysql/mysql-bin.log.index
	relay_log = /var/log/mysql/mysql-relay-bin
	relay_log_index = /var/log/mysql/mysql-relay-bin.index

save & exit.
	
STEP-3: sudo systemctl restart mysql

STEP-4: sudo systemctl status mysql
 
STEP-5: sudo mysql -u root -p

STEP-6: mysql > CREATE USER 'demo1'@'192.168.161.130' IDENTIFIED WITH mysql_native_password BY 'Demo@123';     // 192.168.161.130 = Slave IP

STEP-7: mysql > GRANT REPLICATION SLAVE ON *.* TO 'demo1'@'192.168.161.130';                                  // 192.168.161.130 = Slave IP
 
STEP-8: mysql > FLUSH PRIVILEGES;

STEP-9: mysql> SHOW MASTER STATUS;

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000082 |      857 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)




+------------------+
|    SLAVE-DB2     |    
+------------------+

STEP-1: sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
	
	bin-address  = 192.168.161.130 (Slave IP)
	server-id    = 2

	#######Add at the end of file#############
	log_bin = /var/log/mysql/mysql-bin.log
	log_bin_index =/var/log/mysql/mysql-bin.log.index
	relay_log = /var/log/mysql/mysql-relay-bin
	relay_log_index = /var/log/mysql/mysql-relay-bin.index

save & Exit.
STEP-2: sudo systemctl restart mysql

STEP-3: sudo mysql -u root -p

STEP-4: mysql> stop slave;

STEP-5: mysql>  CHANGE MASTER TO MASTER_HOST ='192.168.161.131', MASTER_USER ='demo1', MASTER_PASSWORD ='Demo@123', MASTER_LOG_FILE = 'mysql-bin.000082', MASTER_LOG_POS = 857, SOURCE_SSL=1;

STEP-6: mysql> START SLAVE;

STEP-7: mysql > SHOW REPLICA STATUS \G;

*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: 192.168.161.131
                  Source_User: khan
                  Source_Port: 3306
                Connect_Retry: 60
              Source_Log_File: mysql-bin.000001
          Read_Source_Log_Pos: 1553
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 326
        Relay_Source_Log_File: mysql-bin.000001
           Replica_IO_Running: Yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 1553
              Relay_Log_Space: 536
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Source_SSL_Allowed: Yes
           Source_SSL_CA_File:
           Source_SSL_CA_Path:
              Source_SSL_Cert:
            Source_SSL_Cipher:
               Source_SSL_Key:
        Seconds_Behind_Source: 0
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Source_Server_Id: 1
                  Source_UUID: 5a3baf26-bffe-11ec-8917-000c29491510
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 86400
                  Source_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Source_SSL_Crl:
           Source_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Source_TLS_Version:
       Source_public_key_path:
        Get_Source_public_key: 0
            Network_Namespace:
--------------------------------------


On Master Machine, create a database;
CREATE DATABASE demo; ---lets verify on slave machine

CREATE TABLE orderz (oid INT NOT NULL AUTO_INCREMENT PRIMARY KEY, orderdate DATE, cid INT);


desc demo1;

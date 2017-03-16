
## Create tables and procedure

* Create vote record memory table:
```sql
DROP TABLE IF EXISTS `vote_record_memory`;
CREATE TABLE `vote_record_memory` (
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    `user_id` varchar(20) NOT NULL DEFAULT '',
    `vote_num` int(10) unsigned NOT NULL DEFAULT '0',
    `group_id` int(10) unsigned NOT NULL DEFAULT '0',
    `status` tinyint(2) unsigned NOT NULL DEFAULT '1',
    `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `index_user_id` (`user_id`) USING HASH
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

* Create vote record table:
```sql
DROP TABLE IF EXISTS `vote_record`;
CREATE TABLE `vote_record` (
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    `user_id` varchar(20) NOT NULL DEFAULT '' COMMENT '用户Id',
    `vote_num` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '投票数',
    `group_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '用户组id 0-未激活用户 1-普通用户 2-vip用户 3-管理员用户',
    `status` tinyint(2) unsigned NOT NULL DEFAULT '1' COMMENT '状态 1-正常 2-已删除',
    `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (`id`),
    KEY `index_user_id` (`user_id`) USING HASH COMMENT '用户ID哈希索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='投票记录表';
```

* Create rand string generate function:
```sql
-- 创建生成长度为n的随机字符串的函数
DELIMITER // -- 修改MySQL delimiter：'//'
DROP FUNCTION IF EXISTS `rand_string` //
SET NAMES utf8 //
CREATE FUNCTION `rand_string` (n INT) RETURNS VARCHAR(255) CHARSET 'utf8'
BEGIN 
    DECLARE char_str varchar(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    DECLARE return_str varchar(255) DEFAULT '';
    DECLARE i INT DEFAULT 0;
    WHILE i < n DO
        SET return_str = concat(return_str, substring(char_str, FLOOR(1 + RAND()*62), 1));
        SET i = i+1;
    END WHILE;
    RETURN return_str;
END //
```

* Create procedure for insert data into `vote_record_memory` table:
```sql
-- 创建插入数据的存储过程
DROP PROCEDURE IF EXISTS `add_vote_record_memory` //
CREATE PROCEDURE `add_vote_record_memory`(IN n INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE vote_num INT DEFAULT 0;
    DECLARE group_id INT DEFAULT 0;
    DECLARE status TINYINT DEFAULT 1;
    WHILE i < n DO
        SET vote_num = FLOOR(1 + RAND() * 10000);
        SET group_id = FLOOR(0 + RAND()*3);
        SET status = FLOOR(1 + RAND()*2);
        INSERT INTO `vote_record_memory` VALUES (NULL, rand_string(20), vote_num, group_id, status, NOW());
        SET i = i + 1;
    END WHILE;
END //
DELIMITER ;  -- 改回默认的 MySQL delimiter：';'
```

## Action start

1. Call procedure above and insert record into memory table (10w records generated need about 40 minutes):
    ```sql
    -- 调用存储过程 生成100W条数据
    CALL add_vote_record_memory(1000000);
    ```
1. Checking record in memory table:
    ```sql
    SELECT count(*) FROM `vote_record_memory`;
    -- count(*)
    -- 105646
    ```
1. Insert record into persist table from memory table (10w records insert need about 13s):
    ```sql
    INSERT INTO vote_record SELECT * FROM `vote_record_memory`;
    ```
1. Checking record in persist table:
    ```sql
    SELECT count(*) FROM `vote_record`;
    -- count(*)
    -- 105646
    ```
1. (optional) Create a procedure for copy record batch:
    ```sql
    -- 参数n是每次要插入的条数
    -- lastid是已导入的最大id
    CREATE PROCEDURE `copy_data_from_tmp`(IN n INT)
    BEGIN
        DECLARE lastid INT DEFAULT 0;
        SELECT MAX(id) INTO lastid FROM `vote_record`;
        INSERT INTO `vote_record` SELECT * FROM `vote_record_memory` where id > lastid LIMIT n;
    END
    ```
1. (optional) Insert record batch through procedure:
    ```sql
    -- 调用存储过程 插入60w条
    CALL copy_data_from_tmp(600000);
    ```



## Create tables

* Create vote record memory table;
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

* Create vote record table;
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

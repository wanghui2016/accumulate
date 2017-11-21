## mysql 主要存储引擎:MyISAM,InnoDb,MEMORY,MERGE
  1 MyISAM:访问速度快,但是不支持事物,不支持外键,对事物完整性没有要求或者使用简单的select,insert语句适合用这种存储引擎.
    支持三种不同存储格式:
    静态表(字段是固定长度)
    动态表
    压缩表(每条记录进行压缩)
    
  2 InnoDb:支持事物操作,效率低些(MySQL5.5以后默认存储引擎)
  
  3 MEMORY:内存中数据创建表,速度快,服务器关闭数据会丢失(表还在),比较适合操作(等于和不等于),
           不适合(between和大于或小于,order by),字段不能使用变长字段(BLOB,TEXT),VARCHAR可以使用
           
  4 MERGE:一组MyISAM表(结构相同)组合,

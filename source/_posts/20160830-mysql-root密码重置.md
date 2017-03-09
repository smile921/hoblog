---
title: mysql  root密码重置
date: 2016-08-30 08:36:57
tags:
   - mysql
---

 新装的mysql 或者用过一段时间的测试机mysql数据库忘了密码，又不想重装。想来应该有办法可以重置密码。
 首先想到mysqladmin 但是尝试在后无果，还是需要密码。经过一番查找尝试还是有高人，废话不多说了，光说不练假把式。看我如何操作

 ```
	/etc/init.d/mysql stop
	mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
	mysql -u root mysql
	update user  set password=password('mysqlroot@') where user='root';
	FLUSH PRIVILEGES;
	/etc/init.d/mysql restart
	mysql -uroot -p
	输入先设置的密码 ok
 ```

 ## python 操作mysql

 ```
#!/bin/python
#coding=utf-8
import MySQLdb
import time
from collections import OrderedDict
from colorama import init, Fore

class Mysql(object):

    '''
        获取当前系统时间 
        2016-08-30 11:13:18
    '''
    def get_current_time(self):
        created_time = time.strftime(
            '[%Y-%m-%d %H:%M:%S]',
            time.localtime(
                time.time()))
        created_time = created_time.split('[')[1]
        created_time = created_time.split(']')[0]
        return created_time

    '''
        host
        user
        password
        db
        port
    '''
    def __init__(self, host, user, passwd, db, port):
        try:
            self.db = MySQLdb.connect(
                host=host,
                user=user,
                passwd=passwd,
                db=db,
                port=port,
                charset='utf8')
            self.cur = self.db.cursor()
        except MySQLdb.Error as e:
            print Fore.RED + '连接数据库失败'
            print Fore.RED + self.get_current_time(), '[%Y-%m-%d %H:%M:%S]', time.localtime(time.time())

    '''
        table 表名称
        my_dict 要插入的数据，一个有序字典
    '''
    def insert_data(self, table, my_dict):
        try:
            cols = ','.join(my_dict.keys())
            values = '","'.join(my_dict.values())
            values = '"' + values + '"'
            try:
              #  print "table:%s,cols:%s,values:%s." %(table, cols, values)
                sql = "insert into %s (%s) values(%s)" % (table, cols, values)
              #  print "sql:",sql
                result = self.cur.execute(sql)
                self.db.commit()
                if result:
                    return 1
                else:
                    return 0
            except MySQLdb.Error as e:
                self.db.rollback()
                if "key 'PRIMARY'" in e.args[1]:
                    print Fore.RED + self.get_current_time(), "数据已存在，未插入数据"
                else:
                    print Fore.RED + self.get_current_time(), "插入数据失败，原因 %d: %s" % (e.args[0], e.args[1])
        except MySQLdb.Error as e:
            print Fore.RED + self.get_current_time(), "数据库错误，原因%d: %s" % (e.args[0], e.args[1])

    def query_data(self,sql):
        try:
            try:
                result = self.cur.execute(sql)
                self.db.commit()
                if result:
                    return 1
                else:
                    return 0
            except MySQLdb.Error as e:
                self.db.rollback()
                if "key 'PRIMARY'" in e.args[1]:
                    print Fore.RED + self.get_current_time(), "数据已存在，未插入数据"
                else:
                    print Fore.RED + self.get_current_time(), "插入数据失败，原因 %d: %s" % (e.args[0], e.args[1])
            pass
        except MySQLdb.Error as e:
            print Fore.RED + self.get_current_time(), "数据库错误，原因%d: %s" % (e.args[0], e.args[1])
            
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'xxx',
        'HOST': '10.x.x5',
        'USER': 'wxspider',
        'PASSWORD': 'wxxxl235',
        'PORT': 3306
    }
}
           
if __name__ == '__main__':
    host = DATABASES['default']['HOST']
    user = DATABASES['default']['USER']
    passwd = DATABASES['default']['PASSWORD']
    db = DATABASES['default']['NAME']
    port = DATABASES['default']['PORT']
    mysql = Mysql(host, user, passwd, db, port)
    created_time = mysql.get_current_time()
    print created_time
    dicts = OrderedDict()
    dicts['id']='2'
    dicts['name']='python'
    tname='test'
    # 测试插入数据
    result = mysql.insert_data(tname, dicts)
    if result:
        print Fore.GREEN + "article_table：数据保存成功！"
    else:
        print Fore.RED + "article_table：数据保存失败！"

    sql = 'select * from test'
    #     测试查询数据
    result = mysql.query_data(sql)
    if result:
        print Fore.GREEN + "：成功！"
        tp = mysql.cur.fetchall()
        print type(tp)
        print tp
    else:
        print Fore.RED + "：失败！"
    print result

 ```


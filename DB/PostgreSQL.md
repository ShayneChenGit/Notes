1. 查看所有分区

  ```sql
  df -h
  ```


2. 进入数据库分区

  ```
  cd /var/app_data/ddei_db
  ```


3. 查看数据库

  ```plsql
  psql ddei sa
  ```


4. 退出数据库

  ```plsql
  \q
  ```


5. 列出所有表

  ```plsql
  \dt
  ```


6. 查看某个表的表结构

  ```plsql
  \d tb_policy
  ```

7. 创建表

   ```sql
   CREATE TABLE table_name (
       column_name1 datatype,
       ... ...
       column_nameN datatype,
       PRIMARY KEY( one or more columns ) 
   );
   ```

8. 修改字段类型

   ```sql
   ALTER TABLE table_name
   ALTER COLUMN column_name [SET DATA] TYPE new_data_type;
   ```

9. 增加字段

   ```sql
   ALTER TABLE table_name
   ADD COLUMN new_column_name data_type;
   ```

10. 增加多个字段

```sql
ALTER TABLE table_name
ADD COLUMN new_column_name_1 data_type constraint,
ADD COLUMN new_column_name_2 data_type constraint,
...
ADD COLUMN new_column_name_n data_type constraint;
```

11. 删除字段

```sql
ALTER TABLE table_name DROP COLUMN column_name;
```

12. 重命名字段

```sql
ALTER TABLE table_name RENAME COLUMN column_name TO new_column_name;
```
13. 插入数据

```sql
INSERT INTO table_name (column_list) VALUES (value_list);
```
14. 插入完整数据(两种方式)

```sql
INSERT INTO products (id, name, price, remark) VALUES (1, 'iPhone', 5028, '');
INSERT INTO products VALUES (2, 'iPad', 2088, '') ;
```
15. 插入部分数据

```sql
INSERT INTO products (id, name, price) VALUES (3, 'iMac', 7088);
```

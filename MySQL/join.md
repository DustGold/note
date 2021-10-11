![1618560768478](C:%5Cnote%5CMySQL%5CUntitled%5Cimg%5C1618560768478.png)

```mysql
FULL OUTER JOIN mysql不支持：
6
select *from tb1_emp a left join tb1_dept b on a.deptId = b.id 
union(合并加去重)
select *from tb1_emp a right join tb1_dept b on a.deptId = b.id
7
select *from tb1_emp a left join tb1_dept b on a.deptId = b.id where b.id is null
union(合并加去重)
select *from tb1_emp a right join tb1_dept b on a.deptId = b.id where a.deptId is null;
```




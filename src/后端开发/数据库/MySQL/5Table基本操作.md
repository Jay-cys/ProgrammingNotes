操作传统关系数据库table的函数已在第二节列出：下面进行详细的使用解析

# 1 Table.insert
```cpp
// Accessing an existing table
var myTable = db.getTable("my_table");
// Insert a row of data.
myTable.insert("id", "name")
	.values(1, "Imani")
	.values(2, "Adam")
	.execute();
```
![image.png](.assets/1608281112092-c4e88737-7d6b-4ea0-ad88-5b7000f63e60.png)

# 2 Table.select
![image.png](.assets/1608281165481-0e6643cf-0233-465c-a6bf-7f52670f000e.png)

# 3 Table.update
![image.png](.assets/1608281197924-854661dc-8855-43aa-bfe9-3f47aed5342f.png)

# 4 Table.delete
![image.png](.assets/1608281220390-1d60e8bc-8ad5-4867-b8a8-ce2bb6ddd4fe.png)


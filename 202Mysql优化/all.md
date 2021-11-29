

# MySql



| 字段          | 含义                      | 示例                           |
| ------------- | ------------------------- | ------------------------------ |
| id            |                           |                                |
| select_type   |                           |                                |
| table         |                           |                                |
| partitions    |                           |                                |
| type          | [访问方法](#Type访问方法) | const、ref、ref_or_null、index |
| Possible_keys |                           |                                |
|               |                           |                                |
|               |                           |                                |



## Type访问方法

`ref` 二级索引列与常数等值比较，采用二级索引来执行查询的访问方法




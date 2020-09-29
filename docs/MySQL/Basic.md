## 数据库范式

1. 第一范式：保证每一列都是原子的，不可再分的。
2. 第二范式：在第一范式的基础上，消除了非主属性对码的部分依赖。
3. 第三范式：在第二范式的基础上，消除了非主属性对码的传递依赖。
4. BC范式：在第三范式的基础上，消除对主码子集的依赖。

## 数据库引擎的对比

[MySQL 8.0 - Chapter 16 Alternative Storage Engines](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html)

| Feature                                    | MyISAM       | Memory           | InnoDB       | Archive      | NDB          |
| ------------------------------------------ | ------------ | ---------------- | ------------ | ------------ | ------------ |
| **B-tree indexes**                         | Yes          | Yes              | Yes          | No           | No           |
| **Clustered indexes**                      | No           | No               | Yes          | No           | No           |
| **Hash indexes**                           | No           | Yes              | No (note 8)  | No           | Yes          |
| **Full-text search indexes**               | Yes          | No               | Yes (note 6) | No           | No           |
| **Index caches**                           | Yes          | N/A              | Yes          | No           | Yes          |
| **Transactions**                           | No           | No               | Yes          | No           | Yes          |
| **MVCC**                                   | No           | No               | Yes          | No           | No           |
| **Locking granularity**                    | Table        | Table            | Row          | Row          | Row          |
| **Foreign key support**                    | No           | No               | Yes          | No           | Yes (note 5) |
| **Data caches**                            | No           | N/A              | Yes          | No           | Yes          |
| **Storage limits**                         | 256TB        | RAM              | 64TB         | None         | 384EB        |
| **Backup/point-in-time recovery** (note 1) | Yes          | Yes              | Yes          | Yes          | Yes          |
| **Cluster database support**               | No           | No               | No           | No           | Yes          |
| **Compressed data**                        | Yes (note 2) | No               | Yes          | Yes          | No           |
| **Encrypted data**                         | Yes (note 3) | Yes (note 3)     | Yes (note 4) | Yes (note 3) | Yes (note 3) |
| **Geospatial data type support**           | Yes          | No               | Yes          | Yes          | Yes          |
| **Geospatial indexing support**            | Yes          | No               | Yes (note 7) | No           | No           |
| **Replication support** (note 1)           | Yes          | Limited (note 9) | Yes          | Yes          | Yes          |
| **T-tree indexes**                         | No           | No               | No           | No           | Yes          |
| **Update statistics for data dictionary**  | Yes          | Yes              | Yes          | Yes          | Yes          |

**Notes:**

1. Implemented in the server, rather than in the storage engine.

2. Compressed MyISAM tables are supported only when using the compressed row format. Tables using the compressed row format with MyISAM are read only.

3. Implemented in the server via encryption functions.

4. Implemented in the server via encryption functions; In MySQL 5.7 and later, data-at-rest tablespace encryption is supported.

5. Support for foreign keys is available in MySQL Cluster NDB 7.3 and later.

6. InnoDB support for FULLTEXT indexes is available in MySQL 5.6 and later.

7. InnoDB support for geospatial indexing is available in MySQL 5.7 and later.

8. InnoDB utilizes hash indexes internally for its Adaptive Hash Index feature.

流程：
1. 查询快照列表 --> ecs:DescribeSnapshots
2. 使用数据盘快照创建云盘 --> ecs:CreateDisk
3. 挂载云盘到备份实例 --> ecs:CreateDisk
4. 备份完后自动卸载实例 --> ecs:DetachDisk
准备：
创建对应权限RAM 账户 获取 key
自定义策略
```json
{
  "Version": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:DescribeSnapshots",
        "ecs:CreateDisk",
        "ecs:AttachDisk",
        "ecs:DetachDisk",
        "ecs:DescribeInstances",
        "ecs:DescribeDisks"
      ],
      "Resource": "*"
    }
  ]
}
```
### 1. 创建新 ECS 实例

- 在阿里云控制台中创建一个新的 ECS 实例，配置与现有数据库实例相匹配的操作系统和规格（建议保持一致，避免兼容性问题）。

### 2. 基于已有快照创建新的云盘

- 选择与原始数据盘相同的类型和容量，保证数据一致性

### 3. 挂载新云盘到新 ECS 实例

- 将新创建的云盘挂载到新 ECS 实例。

- 登录新 ECS 实例，确认系统识别到新挂载的云盘（ `lsblk` ）

==注意== ：此处挂载 需要手动执行 挂载到 /mnt/data 下

```shell
# 1. 查看磁盘挂载情况 
lsblk
=========================================
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  40G  0 disk 
└─vda1 253:1    0  40G  0 part /
vdb    1:16   0  3550G  0 disk  # 待挂载磁盘
=============================================
# 2. 对新磁盘进行分区和格式化 (此处跳过) 如果是新盘（没有 `part` 和文件系统），执行
mkfs -t ext4 /dev/vdb #⚠️ 以上操作会清空该盘数据，若磁盘上已有数据，请 不要执行格式化。

# 3. 创建挂载点目录 
mkdir -p /mnt/data

# 4. 挂载磁盘到目录
mount /dev/vdb /mnt/data

# 5. 验证挂载
df -h | grep /mnt/data
  
```

### 5. 启动 MongoDB 并加载数据盘

- 在新 ECS 实例上安装 MongoDB（版本需与原实例保持一致）。

- 将挂载的云盘挂载到 MongoDB 的数据目录（如 `/mnt/data`）。

- 启动 MongoDB 服务，确认能够正常读取数据盘中的数据库文件。

```shell
mongod --dbpath /mnt/data/db2 --bind_ip 127.0.0.1 --port 27017

  
```

### 6. 使用 `mongodump` 进行数据备份

- 根据需要备份的范围（指定数据库或集合），执行 `mongodump` 命令，例如：

备份指定数据库
**mongodump --db your_database_name --out /backup/path**

备份指定集合

**mongodump --db your_database_name --collection your_collection_name --out /backup/path**


**- `/backup/path` 可以是新实例上的目录，也可以挂载 OSS 或其他存储服务。**

### 7. 验证备份结果

**- 检查 `/backup/path` 下生成的 BSON 和 JSON 文件。**

**- 可使用 `mongorestore` 在测试环境中恢复，确保备份完整。**

  

**---**
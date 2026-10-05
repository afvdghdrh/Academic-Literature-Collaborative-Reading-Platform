## 文献在线交流平台

### 一、背景

查阅科研文献并进行组内交流是实验室一项十分频繁且重要的基础研究工作。

当下，实验室大多是各自从网上搜集文献并在本地批注阅读、依靠微信等方式进行文献的传阅，而要想分享关于自己阅读的看法时多数需要依托于组会等线下形式。由于传统方式各自阅读笔记多数存储在本地，故很难实时充分的共享关于某个方向的想法，也不能很好的保存实验室历届同学的文献阅读积累的知识。

本项目针对传统的实验室文献阅读交流方式不易于实时共享的、不利于传承痛点，拟研发和建立一个面向高校实验室的文献在线交流平台，老师和学生可以上传和分享学术文献、阅读笔记与其他人进行分享和交流。

本系统将每一个文献作为一个条目，并对条目添加分类，在建立一个条目后可以上传多个阅读笔记和多个评论。每个用户均可浏览他人笔记并收藏，也可收藏和下载文献。在此基础上，可以对文献进行统计分析，应用知识图谱找出关联。

---

### 二、需求分析

#### 1. 功能分析（见概要设计-系统结构设计）

#### 2. 技术分析

| 工具            | 用途               | 版本    |
| --------------- | ------------------ | ------- |
| JDK             | Java运行时环境     | 21.0.9  |
| Maven           | 后端构建与依赖管理 | 3.9.4   |
| Spring Boot     | Java Web框架       | 3.5.10  |
| MyBatis-Plus    | ORM框架            | 3.5.5   |
| MySQL           | 关系型数据库       | 8.0.30  |
| Redis           | 缓存、向量数据库   | 8.2.3   |
| 七牛云OSS       | 七牛云OSS服务      | 7.7.0   |
| Knife4j         | 生成接口文档       | 4.4.0   |
| RabbitMQ /Kafka | 消息队列           |         |
| LangChain4j     | 大模型应用框架     |         |
| Node.js         | 前端运行时环境     | 22.14.0 |
| Vue.js          | 前端框架           | 3.5.13  |
| Element-UI      | 前端组件库         | 2.9.6   |
| Axios           | 网络请求库         |         |

---

### 三、概要设计

#### 1. 总体架构

#### 2. 系统结构设计

##### （1）认证模块

- 注册：邮箱（唯一）
- 登录：邮箱 / 用户名（唯一）

##### （2）用户模块

- 个人信息管理
- 上传的文献：分页查询
- 收藏的文献：分页查询
- 发布的笔记：分页查询
- 收藏的笔记：分页查询

##### （3）文献模块

- 文献分类管理
- 上传文献：关联文献分类
- 下载文献
- 查看文献列表：分页条件查询
- 在线阅读
- 收藏文献
- 删除文献：逻辑删除，仅管理员操作

##### （4）笔记模块

- 发布笔记：关联文献
- 审核笔记：仅管理员操作
- 查看笔记列表：分页条件查询
- 查看笔记详情
- 收藏笔记
- 删除笔记：逻辑删除，普通用户仅删除自己发布的笔记

##### （5）评论模块

- 发布评论：关联笔记
- 审核评论：仅管理员操作
- 查看评论列表：级联递归查看
- 点赞评论
- 删除评论：逻辑删除，普通用户仅删除自己发布的评论

##### （6）AI模块

- 文献总结
- 个性化推荐：基于内容（文献分类）、基于用户行为（浏览、收藏、点赞、评论）、基于LLM（需求匹配）
- 智能问答：RAG检索

##### （7）统计模块

- 文献统计分析：浏览量、收藏量、下载量、笔记量
- 笔记统计分析：浏览量、收藏量、评论量
- 文献知识图谱

#### 3. 数据库设计

##### （1）用户信息表（user）

| 字段        | 类型     | 长度 | 索引                        | 描述                     |
| ----------- | -------- | ---- | --------------------------- | ------------------------ |
| id          | BIGINT   | -    | PRIMARY KEY，AUTO_INCREMENT | 用户主键ID               |
| username    | VARCHAR  | 50   | UNIQUE，NOT NUL             | 用户名（唯一）           |
| password    | VARCHAR  | 100  | NOT NULL                    | 加密后的密码             |
| email       | VARCHAR  | 50   | UNIQUE，NOT NUL             | 邮箱（唯一）             |
| avatar      | VARCHAR  | 200  | DEFAULT NULL                | 头像                     |
| grade       | VARCHAR  | 20   | DEFAULT NULL                | 年级                     |
| major       | VARCHAR  | 50   | DEFAULT NULL                | 专业                     |
| role        | TINYINT  | -    | NOT NULL，DEFAULT 0         | 角色：0普通用户，1管理员 |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0-否，1-是     |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

##### （2）文献分类表（category）

| 字段        | 类型    | 长度 | 索引                        | 描述             |
| ----------- | ------- | ---- | --------------------------- | ---------------- |
| id          | BIGINT  | -    | PRIMARY KEY，AUTO_INCREMENT | 分类主键ID |
| name        | VARCHAR | 50   | UNIQUE，NOT NUL             | 分类名称（唯一） |
| description | VARCHAR | 255  | DEFAULT NULL                | 分类描述         |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0-否，1-是     |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

##### （3）文献信息表（paper）

| 字段 | 类型 | 长度 | 索引 | 描述 |
| ---- | ---- | ---- | ---- | ---- |
| id | BIGINT | - | PRIMARY KEY，AUTO_INCREMENT | 文献主键ID |
| title | VARCHAR | 255 | UNIQUE，NOT NUL | 文献标题 |
| title_zh | VARCHAR | 255 | UNIQUE，NOT NUL | 中文文献标题 |
| author_list | VARCHAR | 255 | NOT NULL | 作者列表 |
| journal | VARCHAR | 200 | DEFAULT NULL | 期刊/会议名称 |
| publish_time | DATETIME | - | DEFAULT NULL | 发表日期 |
| category_id | BIGINT | - | UNIQUE，NOT NULL | 文献分类ID |
| abstract | TEXT | - | DEFAULT NULL | 摘要 |
| abstract_zh | TEXT | - | DEFAULT NULL | 中文摘要 |
| keyword_list | VARCHAR | 500 | DEFAULT NULL | 关键词列表 |
| upload_user_id | BIGINT | - | UNIQUE，NOT NUL | 上传用户ID |
| file_url | VARCHAR | 500 | NOT NULL | 文件访问URL |
| view_count | INT | -    | NOT NULL, DEFAULT 0 | 浏览量 |
| download_count | INT | - | NOT NULL, DEFAULT 0 | 下载量 |
| collect_count | INT | - | NOT NULL, DEFAULT 0 | 收藏量 |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0-否，1-是     |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

##### （4）笔记信息表（note）

| 字段 | 类型 | 长度 | 索引 | 描述 |
| ---- | ---- | ---- | ---- | ---- |
| id | BIGINT | - | PRIMARY KEY，AUTO_INCREMENT | 笔记主键ID |
| user_id | BIGINT | - | UNIQUE，NOT NUL | 发布用户ID |
| paper_id | BIGINT | - | UNIQUE，NOT NUL | 关联文献ID |
| title | VARCHAR | 255 | NOT NULL | 笔记标题 |
| content | Text | - | NOT NULL | 笔记内容 |
| view_count | INT | - | NOT NULL, DEFAULT 0 | 浏览量 |
| collect_count | INT | - | NOT NULL, DEFAULT 0 | 收藏量 |
| status | TINYINT | - | NOT NULL, DEFAULT 0 | 审核状态：0-待审核，1-通过，2-拒绝 |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0-否，1-是     |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

##### （5）评论信息表（comment）

| 字段 | 类型 | 长度 | 索引 | 描述 |
| ---- | ---- | ---- | ---- | ---- |
| id | BIGINT | - | PRIMARY KEY，AUTO_INCREMENT | 评论主键ID |
| user_id | BIGINT | - | UNIQUE，NOT NUL | 发布用户ID |
| note_id | BIGINT | - | UNIQUE，NOT NUL | 关联笔记ID |
| parent_id | BIGINT | - | DEFAULT NULL                | 父评论ID，NULL表示顶级评论 |
| content | VARCHAR | - | NOT NULL | 评论内容 |
| like_count | INT | - | NOT NULL, DEFAULT 0 | 点赞量 |
| status | TINYINT | - | NOT NULL, DEFAULT 0 | 审核状态：0-待审核，1-通过，2-拒绝 |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0-否，1-是    |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

##### （6）用户行为表（user_behavior）

| 字段 | 类型 | 长度 | 描述 | 索引 |
| ---- | ---- | ---- | ---- | ---- |
| id | BIGINT | - | PRIMARY KEY，AUTO_INCREMENT | 用户行为主键ID |
| user_id | BIGINT | - | UNIQUE，NOT NUL | 发布用户ID |
| behavior_type | TINYINT | - | NOT NULL | 行为类型：0-浏览，1-下载，2-点赞，3-取消点赞，4-收藏，5-取消收藏 |
| biz_type | TINYINT | - | NOT NULL | 业务类型：0-文献，1-笔记，2-评论 |
| biz_id | BIGINT | - | UNIQUE，NOT NUL | 关联业务ID |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0-否，1-是     |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

##### （7）文献关系表（paper_relation）-后续扩展

| 字段 | 类型 | 长度 | 描述 | 索引 |
| ---- | ---- | ---- | ---- | ---- |
| id | BIGINT | - | PRIMARY KEY，AUTO_INCREMENT | 文献关系主键ID |
| source_id | BIGINT | - | UNIQUE，NOT NUL | 源文献ID |
| target_id | BIGINT | - | UNIQUE，NOT NUL | 目标文献ID |
| relation_type | TINYINT | - | NOT NULL | 关系类型：0-引用，1-相似，2-同作者，3-同关键词 |
| weight | DECIMAL | 5,4 | NOT NUL | 关系权重 |
| is_deleted  | TINYINT  | -    | NOT NULL, DEFAULT 0         | 逻辑删除：0否，1是       |
| create_time | DATETIME | -    | NOT NULL                    | 创建时间                 |
| update_time | DATETIME | -    | NOT NULL                    | 更新时间                 |

---

#### 四、详细设计




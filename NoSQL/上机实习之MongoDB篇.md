# 上机实习之 MongoDB 篇

大数据管理技术 NoSQL MongoDB

---

[TOC]

---

## **MongoDB 的安装**

### Linux 环境

1. 下载安装包并解压，将文件夹移动至指定位置，并将其可执行文件添加到 PATH 中

    ```sh
    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.3.tgz
    tar -zxvf mongodb-linux-x86_64-3.6.3.tgz
    sudo mv mongodb-linux-x86_64-3.6.3/ /usr/local/mongodb
    sudo chmod -R 777 !$
    export PATH=/usr/local/mongodb/bin:$PATH
    ```

2. 创建数据库目录，并启动 MongoDB 服务。注意：如果数据库目录不是/data/db，可以通过 --dbpath 来指定。

    ```sh
    sudo mkdir -p /data/db
    sudo chmod -R 777 !$
    cd /usr/local/mongodb/bin/
    ./mongod
    ```

3. 连接 MongoDB，进入 Shell 模式

    ```sh
    cd /usr/local/mongodb/bin/
    ./mongo
    ```

    显示如下信息表示连接成功：

    ```text
    MongoDB shell version v3.6.3
    connecting to: mongodb://127.0.0.1:27017
    MongoDB server version: 3.6.3
    Welcome to the MongoDB shell.
    ...
    ...
    >
    ```

---

### Windows 环境

1. 下载安装包。在 [MongoDB 官网](https://www.mongodb.com/download-center?jmp=nav#community)，在 CommunityServer 标签页下下载.zip 便携安装包。

2. 下载完成后解压到 c:\mongodb

3. 创建`C:\data\db`文件夹，如果要选别处，可以通过 --dbpath 来指定。打开 cmd(或 powershell)启动 MongoDB 服务。

    ```powershell
    cd "c:\mongodb\bin"
    .\mongod.exe
    ```

4. 连接 MongoDB 服务

    ```powershell
    cd "c:\mongodb\bin"
    .\mongo.exe
    ```

    显示如下信息表示连接成功：
    ` MongoDB shell version v3.6.3 connecting to: mongodb://127.0.0.1:27017 MongoDB server version: 3.6.3 ... ... > `

## **MongoDB 的使用**

### 数据库操作

1. 查询现有所有数据库

    ```mongo
    > show dbs
    admin   0.000GB
    config  0.000GB
    local   0.000GB
    ```

2. 查询当前数据库。系统默认使用 test。
   在上面列表未出现 test 是由于它目前是空的，还未包含数据。

    ```mongo
    > db
    test
    ```

3. 创建数据库

    ```mongo
    > use bigdata
    switched to db bigdata
    ```

4. 写入数据，再查询数据库列表

    ```mongo
    > db.bigdata.insert({"teacher":"LijunChen"})
    WriteResult({ "nInserted" : 1 })
    > show dbs
    admin    0.000GB
    bigdata  0.000GB
    config   0.000GB
    local    0.000GB
    ```

5. 删除当前数据库

    ```mongo
    > db.dropDatabase()
    { "dropped" : "bigdata", "ok" : 1 }
    ```

### 集合操作

1. 在 test 数据库中创建 student 集合

    ```mongo
    > use test
    switched to db test
    > db.createCollection("student")
    { "ok" : 1 }
    ```

2. 创建集合的相关参数

    ```mongo
    > db.createCollection("score", { capped : true, autoIndexId : true, size : 10000, max : 100 } )
    { "ok" : 1 }
    ```

    其中参数的含义如下：

    - capped(true/**false**)：创建固定大小的集合，此项为 true 时需设置 size 的值
    - size：为固定大小集合设置最大值（字节）
    - autoIndexId(true/**false**)：若此项为 true，自动在 \_id 字段创建索引
    - max：设置固定集合中包含文档的最大数量

3. 删除当前集合的某个数据库

    ```mongo
    >db.student.drop()
    true
    ```

### 文档操作

1. 增。向 student 集合加入张三同学的文档

    ```mongo
    > db.student.insert({name:'ZhangSan',major:'CS',score:59,skill:['Redis','MongoDB','Cassandra']})
    WriteResult({ "nInserted" : 1 })
    > db.student.find()
    { "_id" : ObjectId("5ab4c2a228e0721a97dcd394"), "name" : "ZhangSan", "major" : "CS", "score" : 59, "skill" : [ "Redis", "MongoDB", "Cassandra" ] }
    ```

    也可先将李四同学的文档写入一个变量，再插入集合。

    ```mongo
    > document=({name:'LiSi',major:'AI',score:58,skill:['C++','Python','Java']})
    {
    "name" : "LiSi",
    "major" : "AI",
    "score" : 58,
    "skill" : [
    "C++",
    "Python",
    "Java"
    ]
    }
    > db.student.insert(document)
    WriteResult({ "nInserted" : 1 })
    > db.student.find()
    { "_id" : ObjectId("5ab4c2a228e0721a97dcd394"), "name" : "ZhangSan", "major" : "CS", "score" : 59, "skill" : [ "Redis", "MongoDB", "Cassandra" ] }
    { "_id" : ObjectId("5ab4c37d28e0721a97dcd395"), "name" : "LiSi", "major" : "AI", "score" : 58, "skill" : [ "C++", "Python", "Java" ] }
    ```

2. 改。我们将成绩为 59 的同学捞一下，改成 61 分。

    ```mongo
    > db.student.update({'score':59},{$set:{'score':61}})
    WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
    > db.student.find()
    { "_id" : ObjectId("5ab4c2a228e0721a97dcd394"), "name" : "ZhangSan", "major" : "CS", "score" : 61, "skill" : [ "Redis", "MongoDB", "Cassandra" ] }
    { "_id" : ObjectId("5ab4c37d28e0721a97dcd395"), "name" : "LiSi", "major" : "AI", "score" : 58, "skill" : [ "C++", "Python", "Java" ] }
    ```

    可以看到，张三同学从不及格变成了及格。
    注意，上述命令只会影响第一个成绩为 59 的同学。若需要将多个同学从 59 分改成 61 分，需执行以下命令。

    ```mongo
    > db.student.update({'score':59},{$set:{'score':61}},{multi:true})
    ```

3. 删。
   首先在 student 集合中插入两个什么也不会的王五同学

    ```mongo
    > db.student.insert({name:'WangWu',major:'Null',score:0,skill:['Sleep']})
    WriteResult({ "nInserted" : 1 })
    > db.student.insert({name:'WangWu',major:'Null',score:0,skill:['Sleep']})
    WriteResult({ "nInserted" : 1 })
    > db.student.find()
    { "_id" : ObjectId("5ab4c2a228e0721a97dcd394"), "name" : "ZhangSan", "major" : "CS", "score" : 61, "skill" : [ "Redis", "MongoDB", "Cassandra" ] }
    { "_id" : ObjectId("5ab4c37d28e0721a97dcd395"), "name" : "LiSi", "major" : "AI", "score" : 58, "skill" : [ "C++", "Python", "Java" ] }
    { "_id" : ObjectId("5ab4cebf28e0721a97dcd396"), "name" : "WangWu", "major" : "Null", "score" : 0, "skill" : [ "Sleep" ] }
    { "_id" : ObjectId("5ab4cec128e0721a97dcd397"), "name" : "WangWu", "major" : "Null", "score" : 0, "skill" : [ "Sleep" ] }
    ```

    下面将分数为 0 分的同学移出 student 集合。

    ```mongo
    > db.student.remove({"score":0})
    WriteResult({ "nRemoved" : 2 })
    > db.student.find()
    { "_id" : ObjectId("5ab4c2a228e0721a97dcd394"), "name" : "ZhangSan", "major" : "CS", "score" : 61, "skill" : [ "Redis", "MongoDB", "Cassandra" ] }
    { "_id" : ObjectId("5ab4c37d28e0721a97dcd395"), "name" : "LiSi", "major" : "AI", "score" : 58, "skill" : [ "C++", "Python", "Java" ] }
    ```

    可见，两个 0 分同学已被移出集合。
    类似地，上述命令也可写为条件表达式：

    ```mongo
    db.student.remove({"score":{$lt:60}})
    ```

    可将 60 分以下的同学全部移除 4. 查。
    上述操作中 find()已出现过多次，通过 pretty()可将输出格式化。

    ```mongo
    > db.student.find().pretty()
    {
            "_id" : ObjectId("5ab4c2a228e0721a97dcd394"),
            "name" : "ZhangSan",
            "major" : "CS",
            "score" : 61,
            "skill" : [
                    "Redis",
                    "MongoDB",
                    "Cassandra"
            ]
    }
    {
            "_id" : ObjectId("5ab4c37d28e0721a97dcd395"),
            "name" : "LiSi",
            "major" : "AI",
            "score" : 58,
            "skill" : [
                    "C++",
                    "Python",
                    "Java"
            ]
    }
    ```

    条件查询与 SQL 类似。形式为

    ```mongo
    等于：      {<key>:<value>}
    小于：      {<key>:{$lt:<value>}}
    小于等于：  {<key>:{$lte:<value>}}
    大于：      {<key>:{$gt:<value>}}
    大于等于：  {<key>:{$gte:<value>}}
    不等于：    {<key>:{$ne:<value>}}
    ```

    多条件查询命令：

    ```mongo
    AND:    db.student.find({key1:value1, key2:value2}).pretty()
    OR:     db.student.find({$or: [{key1: value1}, {key2:value2}]}).pretty()
    同时：  db.student.find({key1: value1, $or: [{key2:value2},{key3:value3}]}).pretty()
    ```

---

**以下 Java 和 Python 中使用 MongoDB 仅为范例参考，完成上机目标不限定具体语言和工具。**

## **Java 中使用 MongoDB**

1. 首先需要下载驱动包 [mongo-java-driver-3.6.3.jar](http://central.maven.org/maven2/org/mongodb/mongo-java-driver/3.6.3/mongo-java-driver-3.6.3.jar)

2. 在 Java 的 Classpath 中包含该驱动包的路径，或在 IDE 构建项目后导入该包
3. 在你的 Java 项目中运行如下代码(注意:确保你的 MongoDB 服务端正常运行中)

```java
import java.util.ArrayList;
import java.util.List;
import org.bson.Document;
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.model.Filters;
public class TryMongodb{
   public static void main( String args[] ){
        try{
            // 连接到 mongodb 服务
            MongoClient mongoClient = new MongoClient( "localhost" , 27017 );
            // 连接到数据库
            MongoDatabase mongoDatabase = mongoClient.getDatabase("test");
            //创建集合
            mongoDatabase.createCollection("student");
            //选择集合
            MongoCollection<Document> collection = mongoDatabase.getCollection("student");
            //插入文档
            Document document = new Document("name", "ZhangSan").append("major", "CS").append("score", 59);
            List<Document> documents = new ArrayList<Document>();
            documents.add(document);
            collection.insertMany(documents);
            //检索文档
            FindIterable<Document> findIterable = collection.find();
            MongoCursor<Document> mongoCursor = findIterable.iterator();
            while(mongoCursor.hasNext()){
                System.out.println(mongoCursor.next());
            }
            //更新文档
            collection.updateMany(Filters.eq("score", 59), new Document("$set",new Document("likes",61)));
            //删除文档
            //删除符合条件的第一个文档
            collection.deleteOne(Filters.eq("score", 61));
            //删除所有符合条件的文档
            collection.deleteMany (Filters.eq("score", 61));
        }catch(Exception e){
            System.err.println( e.getClass().getName() + ": " + e.getMessage() );
        }
    }
}
```

## **Python 中使用 MongoDB**

1. 安装

    ```sh
    pip3 install pymongo
    ```

2. 在你的 Python 项目中运行如下代码

    ```python
    import pymongo
    from pymongo import MongoClient
    #连接MongoDB服务
    client = MongoClient('localhost',27017)
    #选择数据库及集合
    db = client.test
    student = db.student
    #插入单个文档
    student1 = {"name":"ZhangSan","major":"CS","score":59}
    rs1 = student.insert_one(student1)
    print('One post: {0}'.format(rs1.inserted_id))
    #插入多个文档
    student2 = {"name":"LiSi","major":"AI","score":58}
    student3 = {"name":"WangWu","major":"Null","score":0}
    rs23 = student.insert_many([student2, student3])
    print('Multiple users: {0}'.format(rs23.inserted_ids))
    #检索文档
    student_tmp = student.find_one({"major":"CS"})
    print(student_tmp)
    #条件查询
    rs = student.find({'score':{"$lt":60}}).sort("name")
    for tmp in rs:
        print(tmp)
    #更新文档
    student.update({"name":"ZhangSan"},{'$set':{"score":100}})
    #删除文档
    student.remove({"score": {"$lt":10}})
    ```

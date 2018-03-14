# Mongodb_study

1.nosql
1.1nosql的解释
<1>非关系型数据库；
<2>不是sql
1.2关系型数据库与非关系型数据库的比较
关系型数据库：扩展性差，表结构更改困难；
非关系型数据库：易扩展，数据量大，性能高，灵活，高可用性；
2.mongodb的使用
2.1安装
sudo apt-get install -y mongodb-org
2.2启动服务器的两种方式
sudo service mongod start
sudo mongod --config /etc/mongod.conf
2.3停止服务器
sudo service mongod stop
2.4重启服务器
sudo service mongod restart
2.5检查是否启动成功
ps aux|grep mongod
2.6配置文件的位置
/etc/mongod.conf
2.7默认端口：27017
2.8默认日志位置
/var/log/mongodb/mongod.log
3.客户端启动
mongo
4.退出
exit
5.数据库命令
查看当前数据库：db
查看所有数据库：show dbs/show databases;
切换数据库：use dbname
删除当前数据库：db.dropDatabase()
6.集合的命令
6.1集合的创建
自动创建集合：向不存在的集合中第一次加入数据时，集合就会被创建
手动创建：
db.createCollection(col_name)
6.2查看集合：show collections
6.3删除集合：db.col_name.drop()
7.数据库的增删改查
7.1插入
db.col_name.insert({-id:xxx,键值对1，键值对2...}),注：当主键存在则会报错；
db.col_name.save({-id:xxx,键值对1，键值对2...})，主键存在则会更新值，不存在则会新增；
db.t1.insert/save({_id:111,name:'jack',age:20,gender:true})
7.2更新
将name为jack的人的年龄改为20
db.t1.update({name:'jack'},{age:20})
# 效果，如果原纪录都还有其他字段都会被删除，只会保留_id字段和age字段
db.t1.update({name:'jack'},{$set:{age:20}})
# 更新一条记录，其他字段不会被删除
db.t1.update({},{$set:{}},{multi:true})
# 所有符合条件的数据都会被更新
7.3删除
db.t1.remove(条件)  # 删除所有符合的数据
db.t1.remove(条件，{justOne:true})  # 只删除一条数据
7.4查询
db.t1.find() # 返回符合条件的数据
db.t1.findOne() # 返回符推荐的第一条数据
db.t1.find().pretty()  # 将查询结果格式化输出
8.高级查询
8.1比较运算符
小于/等于：$lt/$lte
大于/等于：$gt/$gte
不等于:$ne
# 查询年龄大于18的人
db.t1.find({age:{$gte:18}})
8.2逻辑运算符
and:在多个条件之间用‘，’即可
# 查询年龄大于或等于18，并且性别为true的人
db.t1.find({age:{$gte:18},gender:true})
or:find({$or:[{条件1},{条件2}]})
# 查询年龄大于18， 或性别为false的学生
db.t1.find({$or:[{age:{$gte:18}},{gender:false}]})
# 查询年龄⼤于18或性别为男⽣， 并且姓名是郭靖
db.t1.find({$or:[{age:{$gt:18},{gender:true}],name:'gj'})
8.3范围运算符-$in/$nin
# 查询年龄为18、 28的学生
db.t1.find({age:{$in:[18,28]}})
8.4正则查询
# 查询姓黄的学生
db.stu.find(name:/^黄/)
db.stu.find(name:{$regex:'^黄'})
8.5limit-限制条数和skip-跳过条数
db.stu.find().limit(num).skip(num)

8.6投影-在查询的结果，选择要显示的字段，值为1表示显示，0表示不显示
注：_id默认为显示
# 查询到的结果直线式姓名和性别
db.stu.find({条件}，{_id:0,name:1,gender:1})
8.7排序-sort()
参数为1升序，参数为-1降序
# 根据性别排序，再根据年龄排序
db.stu.find().sort({gender:1,age:1})
8.8统计个数-count()
db.stu.find({条件}).count();
db.stu.count({条件})；
# 统计性别为男的人数
db.stu.find({gender:true}).count()
# 统计男性并且大于20岁的人数
db.stu.find({age:{$gt:20},gender:true})
8.9去重-distinct()
注;返回的结果为数组
db.col_name.distinct('去重字段'，条件)
# 大于18岁的任都来自哪些地方？（对hometown进行去重）
db.stu.distinct('hometown',{age:{$gt:18}})

9.聚合-aggregate
形式：db.col.aggregate({管道1：{表达式}}，{管道2：{表达式}}.......)
管道：$group,$match,$project,$sort,$limit,%skip,$unwind
表达式：‘$列名’
$sum,$avg,$min.$max,$push,$first,$last
9.1分组-$group
将集合中的数据进行分组，再进行统计
# 统计男生女生各有多少人？
db.stu.aggregate({$group:{_id:'$gender',counter:{$sum:1}}})
将集合中的所有文档分为一组
# 统计学生总人数，平均年龄
db.stu.aggregate(
    {$group:
     {
         _id:null,
         counter:{$sum:1},
         avgAge:{$avg:'$age'}
     }
    }
)
9.2数据透视
# 统计不同性别的学生的姓名
db.stu.aggregate({$group:{_id:'$gender',name:{$push:'$name'}}})
注：待做ppt37的练习题
# 统计出每个country/province下的userid的数量（同一个userid只统计一次）
#先根据userid分组去重
db.tv3.aggregate(
    {$group:{_id:{country:'$country',province:'$province',userid:'$userid'}}},
    {$group:{_id:{country:'$country',province:'$province',userid:'$userid'},count:{$sum:1}}},
)
9.3$match
mach与find的区别，match是管道命令，能将结果交给后面一个管道，find不可以；
# 查询年龄大于20的学生
db.stu.aggregate({$match:{age:{$gt:20}}})
# 查询年龄大于18的男生/女生人数
db.stu.aggregate(
    {$match:{age:{$gt:20}}},
    {$group:{_id:'$gender',counter:{$sum:1}}}
                )
9.4$project
修改输入文档的结构，如重命名，增加删除字段，创建计算结果
# 查询学生的姓名，年龄
db.stu.aggregate({$project:{_id:0,name:0,age:0}})

# 查询男生/女生人数，输出人数
db.stu.aggregate({$group:{_id:'$gender',counter:{$sum:1}}},{$project:{_id:0,counter:1}})
# 统计出每个coutry/province下的userid的数量（同一个userid统计一次），结果中的字段为{country:'**',province:'**',counter:'**'}
db.tv3.aggregate({$group:{_id:{counrty:'$country',province:'$province',userid:'$userid'}}},
                 {$group:{_id:{contry:'$_id.contry',province:'$_id.province',userid:'$_id.userid'},counter:{$sum:1}}},
                 {$project:{_id:0,country:1,province:1,counter:1}}
                )
9.5$sort
将文档排序后输出
# 查询学生信息，按年龄升序
db.stu.aggregate({$sort:{age:1}})
# 查询男生，女生人数，按人数降序
db.stu.aggregate({$group:{_id:'$gender',counter:{$sum:1}}},
                 {$sort:{counter:-1}}
                )
9.6$limit,$skip
$limit限制聚合管道返回的文档数
$skip跳过指定数量的文档，并返回剩余的文档
9.7$unwind
将文档中的某个数组型字段拆分成多条，每条含数组中的一个值
db.col.aggregate({$unwind:'$字段名'})
# {_id:1,item:'t-shirt',size:['S','M','L']}
db.t2.aggregate({$unwind:'$size'})
属性值为preserverNullAndEmptyArrays:true，表示保留属性值为空的文档，为了防止数据丢失
db.t1.aggregate({
$unwind:{
    path:'$字段名'，
    preserverNullAndEmptyArrays:true}
})
10.索引
10.1创建索引-提升查询速度
10.1.1创建唯一索引
db.t1.ensureIndex({'name':1},{'unique':true})
创建唯一索引并消除重复
db.t1.ensureIndex({'name':1},{'unique':true，'dropDups':true})
10.2建立联合索引
db.t1.ensureIndex({name:1,age:1})
10.3查看当前所有索引
db.t1.getIndexes()
10.4删除索引
db.t1.dropIndex('索引名')
11.数据的备份和恢复
11.1备份
mongodump -h host -d dbname -o path
11.2恢复
mongorestore -h host -d dbname --dir path

12mongodb与python的交互
安装pymongo
# 第一步-导包
from pymongo import MongoClient
# 第二步-实例化client
client = MongoClient(host='',port=27017)
# 运用实例对象client进行增删改查的操作

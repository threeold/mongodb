# mongodb基本语法
### 1、排序、条数
    db.getCollection('browses').find({"umeng_device" : "Ahlx_YiLlCnPfFLiHnQ0oWQdGbPI_YsAMlgfbuIMTsDa"}).sort({"c_time":1}).limit(10)



db.getCollection('browses').find({"uid" : 704536,"website" : "mapi","c_time" : 1509432094}).sort({"c_time":-1})


3、group by 的用法

db.getCollection('browses').aggregate([
    {$match : {uid:704536,activity_id:{$gt:"0",$lte:"288"}}},
    {$group : {_id : "$activity_id", nums : {$sum : 1}}},
    {$sort:{nums:-1}}
    ]
    )



db.getCollection('browses').find({"c_time":{$gt:1509958800}}).count();




4、多字段分组
db.getCollection('browses').group(
   {
     key: { version: 1, 'platform': 1 },
     cond: { uid: { $gt: 0} },
     reduce: function( curr, result ) {
                 result.total += 1;
             },
     initial: { total : 0 }
   }
)


5、连表、分组查询

db.getCollection('browses').aggregate([
	{$lookup: {
		from: "test",
		localField: "opact",
		foreignField: "opact",
		as: "test"
 	}}
        ,{$match : {"test.opactname":"首页账户","uid":{$gt:0}}},
        {$group : {_id : "$uid",nums : {$sum : 1}}},
])

6、多条件分组查询

db.getCollection('browses').aggregate([
	{$lookup: {
		from: "test",
		localField: "opact",
		foreignField: "opact",
		as: "test"
 	}}
        ,{$match : {"test.opactname":"首页账户","uid":{$gt:0}}},
        {$group : {_id : {_id: "$version", company: "$test.opactname"},nums : {$sum : 1}}},
])


6、db.users.aggregate([
	{$unwind: "$acceptOrder"},
	{$project: {_id: "$_id", acceptOrder: "$acceptOrder"}},
	{$lookup: {
		from: "orders",
		localField: "acceptOrder",
		foreignField: "_id",
		as: "orders"
	}},
	{$unwind: "$orders"},
	{$project: {_id: "$_id", haveNo: "$orders.haveNo"}},
	{$unwind: "$haveNo"},
	{$lookup: {
		from: "rechargeorders",
		localField: "haveNo",
		foreignField: "_simcardId",
		as: "rechargeOrders"
	}},
	{$unwind: "$rechargeOrders"},
	{$match: { "rechargeOrders.paid": true, "rechargeOrders.createdAt": {$lt: ISODate("2017-01-11T00:00:00.000+08:00") }}},
	{$project:  {_id: "$_id", fee: "$rechargeOrders.fee"}},
	{$group: {
		_id: {_id: "$_id", company: "$company"},
		fee: {$sum: "$fee"},
		count: {$sum: 1}
	}}
])


分组带if的语句
db.getCollection('order').aggregate([
        {$match : {"uid":7,"status":2}},        
        {$group : {_id : {uid: "$uid", "future_id": "$future_id"},"symbol": {$max:"$symbol"},nums : {$sum : {$cond: { if: { $eq: ['$order_type',1]}, then: "$nums", else: 0 }}}
        ,sell_num : {$sum : {$cond: { if:{ $eq: ['$order_type',2]}, then: "$nums", else: 0 }}}
        }}, 
])


if语句多个条件



db.getCollection('order_2019').aggregate([
        {$match : {"status":3}},        
        {$group : {
            _id : {uid: "$uid"},
            uid : {$max :"$uid"},
            nums : {$max :"$nums"},
            total_lot: {$sum :"$nums"},
            total_order_num: {$sum: 1},
            long_num : {$sum : {$cond: { if:{ $eq: ['$order_type',1]}, then: "$nums", else: 0 }}},
            short_num : {$sum : {$cond: { if:{ $eq: ['$order_type',2]}, then: "$nums", else: 0 }}},
            profit_order_num:{$sum : {$cond: { if:{ $gt: ['$profit',0]}, then: "$nums", else: 0 }}},
            loss_order_num:{$sum : {$cond: { if:{ $lt: ['$profit',0]}, then: "$nums", else: 0 }}},
            total_order_time:{$sum: {$subtract:['$close_time','$open_time']}},
            long_order_num: {$sum : {$cond: { if:{$and:[{$eq: ['$order_type',1]},{$gt: ['$profit',0]}]}, then: 1, else: 0 }}},
            short_order_num: {$sum : {$cond: { if:{$and:[{$eq: ['$order_type',2]},{$gt: ['$profit',0]}]}, then: 1, else: 0 }}},
            first_trade_time: {$min :"$open_time"},
            margin : {$sum :"$margin"},
            commission : {$sum :"$commission"},
            coupon_money : {$sum :"$coupon_money"},
            profit : {$sum :"$profit"}
        }}
])





7、distinct的用法

db.getCollection('browses').distinct("umeng_device",{version:"4_3_0"})



8、distinct、count 的用法:
db.runCommand({"distinct":"browses","key":"umeng_device",query:{app_ver:"4_2_0",}}).values.length;



登录控制台，建唯一索引(后台创建唯一的复合索引)：
db.t_user_score.ensureIndex({“userId”:1},{background:1,unique:1});

–查看索引
db.t_user_score.getIndexes()

–查看索引大小
db.t_user_score.totalIndexSize();
–重建索引
db.t_user_score.reIndex()
–删除索引
–db.t_user_score.dropIndex(“id_1”)

–使用索引的结果
db.t_user_score.find({“userId”:1000}).explain();


修改  

db.getCollection('order_2019').find().forEach(function(item) {
   db.getCollection('order_2019').update({"_id":item._id},{$set:{"settle_profit": item.profit}})
});



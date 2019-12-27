---
layout: post
title: Mongoose(MongoDB)简单用法
---

`
接触 nodeJS 有一段时间，记录下常用的 MongoDB 语法，
我使用的是 EggJS 框架，然后安装了 egg-mongoose 插件，
以下谈到的语法是在 mongoose 上的使用，包括 mongoose 的函数和 mongoDB的函数。不做区分。
`

<h4>定义模型</h4>

```js
'use strict';

module.exports = app => {
  const mongoose = app.mongoose;
  const Schema = mongoose.Schema;
  const conn = app.mongooseDB.get('back');
  // new Schema 第一个对象是 model，第二个对象是设置
  const UserSchema = new Schema({
    name: { type: String },
    // 唯一字段，即该字段不能重复
    account: { type: String, unique: true }, 、
    password: { type: String },
    remark: { type: String },
    status: { type: Number },
    create_date: { type: Date, default: Date.now },
    update_date: { type: Date, default: Date.now },
  }, {
    // 支持旧版的 $pushAll
    usePushEach: true,
    // 该设置会在文档更新和创建的时候自动更新 create_date 和 update_date 字段的值。
    timestamps: { createdAt: 'create_date', updatedAt: 'update_date' },
  });

  return conn.model('User', UserSchema);
};

```
> 常用方法。
Model.deleteMany() 
Model.deleteOne() 
Model.find() 
Model.findById() 
Model.findByIdAndDelete() 
Model.findByIdAndRemove() 
Model.findByIdAndUpdate() 
Model.findOne() 
Model.findOneAndDelete() 
Model.findOneAndRemove() 
Model.findOneAndReplace() 
Model.findOneAndUpdate() 
Model.replaceOne() 
Model.updateMany() 
Model.updateOne() 


<h4>增</h4>

```js
// 缺少某些字段也能创建成功，但唯独不能缺少 account，因为它是 unique: true
const result = await this.ctx.model.AuthUser.create(
  {
    name: 'px',
    avatar: 'https://avatars2.githubusercontent.com/u/11468927',
    account: 'px0078',
    password: crypto
      .createHash('md5')
      .update('my password')
      .digest('hex'),,
  }
);
```

<h4>删</h4>

```js
/* 
* mongoDB 会为每一条数据自动生成 _id 字段，它是唯一而且按照时间戳生成，
* 所以删除的时候可以通过查找唯一的 _id 删除。
*/
try {
  await this.ctx.model.AuthUser.remove(
    { _id: '5ae18d0efbbe77641966cb60' },
  ).exec();
} catch (err) {
  this.ctx.logger.error(err.message);
  return '';
}
```

<h4>改</h4>

```js
try {
  // 第一个为查询参数对象 query
  return await this.ctx.model.AuthGroup.findOneAndUpdate(
    { _id: '5ae18d0efbbe77641966cb60' },
    // 第二个为更新的对象
    newData,
    // 第三个为设置
    {
      // newData 里的属性，如果旧数据没有，则会新增该属性。
      new: true,
      // 默认 false，如果该 model 定义了校验函数，则会执行校验
      runValidators: true,
    }
  ).exec();
} catch (err) {
  this.ctx.logger.error(err.message);
  return '';
}
```

<h4>查</h4>

```js
const pageNo = 1;
const pageSize = 10;
const ids = [
  '5ae18d0efbbe77641966cb60',
  '5ae18d0efbbe77641966cb61'
];
return ({
  list: await this.ctx.model.AuthUser.find({
    _id: { $in: ids },
  })
    .skip((pageNo - 1) * pageSize)
    // 按照时间倒序排序
    .sort({ _id: -1 })
    .limit(Number(pageSize))
    // 筛选需要的字段
    .select('avatar account mobile')
    .exec(),
  total: await this.ctx.model.AuthUser.find(query).countDocuments(),
});
```

<h4>高级查询</h4>

```js
/* 
* $all 这个操作符跟 SQL 语法的 in 类似，但不同的是,
*  in 只需满足[]内的某一个值即可, 而 $all 必须满足[]内的所有值
*/
await db.store.find({foods : {$all : ['banana', 'apple']}})

// 查询所有存在age 字段的记录
db.users.find({age: {$exists: true}});

// 查询所有不存在name 字段的记录
db.users.find({name: {$exists: false}});

// $in 包含
// $nin 不包含
// 小于 $lt
// 小于或等于 $lte
// 大于 $gt
// 大于或等于 $gte
// 不等于 $ne
// 或 $or

// likes > 50 && (age === 20 || name === 'px')
db.users.find({
  likes: { $gt: 50 },
  $or: [
    {age: 20}, {name: 'px'}
  ]
})


// $query 查询函数返回布尔值，使用这个方法会把文档转为 JavaScript 对象，会稍微影响性能
db.users.find({
  $query: (obj) => {
    return obj.a.b === 1
  } 
})

// $regex 正则查询，多用于模糊匹配
const { keyword } = ctx.request.query // 参数
const reg = new RegExp(keyword, 'i') // 不区分大小写
const result = await User.find(
  {
    $or : [ // 多条件，数组
      {nick : {$regex : reg}},
      {email : {$regex : reg}}
    ]
  }
)

```

<h4>数组操作（Array Update Operators）</h4>

```js
// 文档地址 https://docs.mongodb.com/manual/reference/operator/update-array/

/* 
* $size 数组元素个数, 可以用它查询特定长度的数组
* favorite_number.length === 3
*/ 
db.users.find({favorite_number: {$size: 3}})

$pop
/*
* 
* The $pop operator removes the first or last element of an array. 
* Pass $pop a value of -1 to remove the first element of an array and 1 to remove the last element in an array.
*/

$pull
/*
* The $pull operator removes from an existing array all instances of a value or values that match a specified condition.
*/
// 删除用户组集合中与此用户相关的数据
// 不加 await ，该语句或不执行（可能因为进程已结束）
await this.ctx.model.AuthGroup.updateMany(
  {},
  {
    $pull: { users: { $in: ['userId'] } },
  },
);

$push
/*
* The $push operator appends a specified value to an array.
*/

```
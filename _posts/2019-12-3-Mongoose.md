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
    account: { type: String, unique: true }, // 唯一字段，即该字段不能重复
    password: { type: String },
    remark: { type: String },
    status: { type: Number },
    create_date: { type: Date, default: Date.now },
    update_date: { type: Date, default: Date.now },
  }, {
    usePushEach: true, // 支持旧版的 $pushAll
    timestamps: { createdAt: 'create_date', updatedAt: 'update_date' }, // 该设置会在文档更新和创建的时候自动更新 create_date 和 update_date 字段的值。
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
// 缺少某些字段也能创建成功，但唯独不能缺少 account
```

<h4>删</h4>

```js
try {
  await this.ctx.model.AuthUser.remove(
    { _id: '5ae18d0efbbe77641966cb60' },
  ).exec();
} catch (err) {
  this.ctx.logger.error(err.message);
  return '';
}
// mongoDB 会未每一条数据自动生成 _id 字段，它是唯一而且按照时间戳生成，所以删除的时候可以通过查找唯一的 _id 删除。
```

<h4>改</h4>

```js
try {
  return await this.ctx.model.AuthGroup.findOneAndUpdate({ _id: '5ae18d0efbbe77641966cb60' }, // 第一个为查询参数对象 query
  newData, // 第二个为更新的对象
  { // 第三个为设置
    new: true, // newData 里的属性，如果旧数据没有，则会新增该属性。
    runValidators: true, // 默认 false，如果该 model 定义了校验函数，则会执行校验
  }).exec();
} catch (err) {
  this.ctx.logger.error(err.message);
  return '';
}
```

<h4>查</h4>

```js
const pageNo = 1;
const pageSize = 10;
const ids = ['5ae18d0efbbe77641966cb60', '5ae18d0efbbe77641966cb61'];
return ({
  list: await this.ctx.model.AuthUser.find({
    _id: { $in: ids },
  })
    .skip((pageNo - 1) * pageSize)
    .sort({ _id: -1 }) // 按照时间倒序排序
    .limit(Number(pageSize))
    .select('avatar account mobile') // 筛选需要的字段
    .exec(),
  total: await this.ctx.model.AuthUser.find(query).countDocuments(),
});
```

<h4>高级查询</h4>

```js
// $all 这个操作符跟 SQL 语法的 in 类似，但不同的是, in 只需满足[]内的某一个值即可, 而 $all 必须满足[]内的所有值
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
db.users.find({
  likes: { $gt: 50 },
  $or: [
    {age: 20}, {name: 'px'}
  ]
}) // likes > 50 && (age === 20 || name === 'px')


// $size 数组元素个数, 可以用它查询特定长度的数组
db.users.find({favorite_number: {$size: 3}}) // favorite_number.length === 3

// $query 查询函数返回布尔值，使用这个方法会把文档转为 JavaScript 对象，会稍微影响性能
db.users.find({
  $query: (obj) => {
    return obj.a.b === 1
  } 
})

```
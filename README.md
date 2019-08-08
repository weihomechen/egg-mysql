# @ifun/egg-mysql

[![Build Status][build-image]][build-url]
[![NPM Version][npm-image]][npm-url]

[build-image]: https://travis-ci.com/weihomechen/egg-mysql.svg?branch=master
[build-url]: https://travis-ci.com/weihomechen/egg-mysql
[npm-image]: https://img.shields.io/npm/v/@ifun/egg-mysql.svg?style=flat-square
[npm-url]: https://www.npmjs.com/package/@ifun/egg-mysql

Forked from [eggjs/egg-mysql](https://github.com/eggjs/egg-mysql)

添加了`index.d.ts`，只为了在`egg-ts`里能不报错跑起来，极简的用了`any`来声明

使用时：

```js
// egg plugin config file with ts
mysql: {
  enable: true,
  package: '@ifun/egg-mysql',
},
```

其他使用和原本的`egg-mysql`一模一样，源于提了缺少声明文件的问题和PR，`egg-mysql`维护人员说有空会加，过了好久也没加，自己随便搞了个。

Aliyun rds client(support mysql portocal) for egg framework

## Install

```bash
$ npm i @ifun/egg-mysql --save
```

MySQL Plugin for egg, support egg application access to MySQL database.

This plugin based on [ali-rds](https://github.com/ali-sdk/ali-rds), if you want to know specific usage, you should refer to the document of [ali-rds](https://github.com/ali-sdk/ali-rds).

## Configuration

Change `${app_root}/config/plugin.js` to enable MySQL plugin:

```js
exports.mysql = {
  enable: true,
  package: '@ifun/egg-mysql',
};
```

Configure database information in `${app_root}/config/config.default.js`:

### Simple database instance

```js
exports.mysql = {
  // database configuration
  client: {
    // host
    host: 'mysql.com',
    // port
    port: '3306',
    // username
    user: 'test_user',
    // password
    password: 'test_password',
    // database
    database: 'test',    
  },
  // load into app, default is open
  app: true,
  // load into agent, default is close
  agent: false,
};
```

Usage:

```js
app.mysql.query(sql, values); // you can access to simple database instance by using app.mysql.
```


### Multiple database instance

```js
exports.mysql = {
  clients: {
    // clientId, access the client instance by app.mysql.get('clientId')
    db1: {
      // host
      host: 'mysql.com',
      // port
      port: '3306',
      // username
      user: 'test_user',
      // password
      password: 'test_password',
      // database
      database: 'test',
    },
    // ...
  },
  // default configuration for all databases
  default: {

  },

  // load into app, default is open
  app: true,
  // load into agent, default is close
  agent: false,
};
```

Usage:

```js
const client1 = app.mysql.get('db1');
client1.query(sql, values);

const client2 = app.mysql.get('db2');
client2.query(sql, values);
```

## CRUD user guide

### Create

```js
// insert
const result = yield app.mysql.insert('posts', { title: 'Hello World' });
const insertSuccess = result.affectedRows === 1;
```

### Read

```js
// get
const post = yield app.mysql.get('posts', { id: 12 });
// query
const results = yield app.mysql.select('posts',{
  where: { status: 'draft' },
  orders: [['created_at','desc'], ['id','desc']],
  limit: 10,
  offset: 0
});
```

### Update

```js
// update by primary key ID, and refresh
const row = {
  id: 123,
  name: 'fengmk2',
  otherField: 'other field value',
  modifiedAt: app.mysql.literals.now, // `now()` on db server
};
const result = yield app.mysql.update('posts', row);
const updateSuccess = result.affectedRows === 1;
```

### Delete

```js
const result = yield app.mysql.delete('table-name', {
  name: 'fengmk2'
});
```

## Transaction

### Manual control

- adventage: ```beginTransaction```, ```commit``` or ```rollback``` can be completely under control by developer
- disadventage: more handwritten code, Forgot catching error or cleanup will lead to serious bug.

```js
const conn = yield app.mysql.beginTransaction();

try {
  yield conn.insert(table, row1);
  yield conn.update(table, row2);
  yield conn.commit();
} catch (err) {
  // error, rollback
  yield conn.rollback(); // rollback call won't throw err
  throw err;
}
```

###  Automatic control: Transaction with scope

- API：`*beginTransactionScope(scope, ctx)`
  - `scope`: A generatorFunction which will execute all sqls of this transaction.
  - `ctx`: The context object of current request, it will ensures that even in the case of a nested transaction, there is only one active transaction in a request at the same time.
- adventage: easy to use, as if there is no transaction in your code.
- disadvantage: all transation will be successful or failed, cannot control precisely

```js
const result = yield app.mysql.beginTransactionScope(function* (conn) {
  // don't commit or rollback by yourself
  yield conn.insert(table, row1);
  yield conn.update(table, row2);
  return { success: true };
}, ctx); // ctx is the context of current request, access by `this.ctx`.
// if error throw on scope, will auto rollback
```

## Advance

### Custom SQL splicing

```js
const results = yield app.mysql.query('update posts set hits = (hits + ?) where id = ?', [1, postId]);
```

### Literal

If you want to call literals or functions in mysql , you can use `Literal`.

#### Inner Literal
- NOW(): The database system time, you can obtain by `app.mysql.literals.now`.

```js
yield app.mysql.insert(table, {
  create_time: app.mysql.literals.now
});

// INSERT INTO `$table`(`create_time`) VALUES(NOW())
```

#### Custom literal

The following demo showed how to call `CONCAT(s1, ...sn)` funtion in mysql to do string splicing.

```js
const Literal = app.mysql.literals.Literal;
const first = 'James';
const last = 'Bond';
yield app.mysql.insert(table, {
  id: 123,
  fullname: new Literal(`CONCAT("${first}", "${last}"`),
});

// INSERT INTO `$table`(`id`, `fullname`) VALUES(123, CONCAT("James", "Bond"))
```

## Questions & Suggestions

Please open an issue [here](https://github.com/eggjs/egg/issues).

## License

[MIT](LICENSE)

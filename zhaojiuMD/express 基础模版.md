```javascript

const express = require('express')

const app = express();

const port = 9000;

let MongoClient = require('mongodb').MongoClient;

const bodyParser = require('body-parser');

const router = express.Router();

let linuxDBClient = null;

  

const linuxDBIp = 'mongodb://127.0.0.1:27017/testdb';

  

app.all('*', function (req, res, next) {

res.header('Access-Control-Allow-Origin', '*');

res.header('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild');

res.header('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS,HEAD');

  

if (req.method === 'OPTIONS') {

res.send(200);

} else {

next();

}

});

  

app.use(bodyParser.json({ limit: '50mb' }));

app.use(bodyParser.urlencoded({ limit: '50mb', extended: true }));

app.use('/test', router);

  

(async () => {

  

// 创建并保存MongoDB客户端实例

linuxDBClient = new MongoClient(linuxDBIp);

  

// 连接数据库

await linuxDBClient.connect();

  

let linuxDB = linuxDBClient.db('test');

  

console.log('数据库连接成功');

  

app.use((req, res, next) => {

req.linuxDB = linuxDB;

next();

});

// router.use('/',(req, res) => { res.send('Hello World!') });

app.listen(port, () => {

console.log(`Example app listening on port ${port}`)

})

  

})();

  

// 关闭数据库连接的函数

async function closeConnections() {

console.log('正在关闭数据库连接...');

try {

if (linuxDBClient) {

await linuxDBClient.close();

linuxDBClient = null;

console.log('Linux数据库连接已关闭');

}

} catch (error) {

console.error('关闭数据库连接时出错:', error);

}

}

  

// 监听进程退出信号

process.on('SIGINT', async () => {

console.log('收到SIGINT信号');

await closeConnections();

process.exit(0);

});

  

process.on('SIGTERM', async () => {

console.log('收到SIGTERM信号');

await closeConnections();

process.exit(0);

});

  

// 处理进程退出

process.on('exit', (code) => {

// console.log(`进程即将退出，退出码: ${code}`);

});

  

process.on('uncaughtException', async (error) => {

console.error('未捕获的异常:', error);

});

  

process.on('unhandledRejection', async (reason, promise) => {

console.error('未处理的拒绝:', reason);

});
```

可以直连本地测试mongodb数据库
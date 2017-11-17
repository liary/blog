
# async 函数传入Promise.all中表现
> 最近用node做服务端渲染，有人建议我Promise.all里只接收Promise对象而非async函数，原由是async会串行执行，这明显不符合我对Promise.all的理解

## 实验方案
1. 分别执行Promise.all传入Promise对象/async函数的函数
2. 标记各组用时
3. 得出结论

## 代码
```javascript
// test async all
const cr = (time, id) => {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve({id, time})
		}, time)
	})
}
const as = (time, id) => {
	return async function() {
		const a = await cr(time, id);
		return a;
	}
}
const sas = async function() {
	let time = new Date();
	console.log('async ==>')
	console.log('async ==>start time:', time);
	const res = await Promise.all([as(500, 43)(), as(600, 4353)(), as(1000,32222)()]);
	time = time.getTime() - new Date().getTime();
	console.log('async ==>end time:', time, new Date());
	console.log(res)
}
const sp = async function() {
	console.log('promise ==>')
	let time = new Date();
	console.log('promise ==>start time:', time);
	const res = await Promise.all([cr(500, 43), cr(600, 4353), cr(1000, 32222)]);
	time = time.getTime() - new Date().getTime();
	console.log('promise ==>end time:', time, new Date());
	console.log(res)
}
sas();
sp();
```

## 结果
```javascript
async ==>
async ==>start time: ***
promise ==>
promise ==>start time: ***
promise ==>end time: -1001 ***
[ { id: 43, time: 500 },
  { id: 4353, time: 600 },
  { id: 32222, time: 1000 } ]
async ==>end time: -1007 ***
[ { id: 43, time: 500 },
  { id: 4353, time: 600 },
  { id: 32222, time: 1000 } ]
```

## 结论
promise和async用时相差不大，基本可以说明async传进Promise.all也是并发的，所以用Promise.all不用太纠结传入的是否Promise对象，反正两者都会并发请求的😁
> 我偷偷的把代码执行的时间给注释了，以防我熬夜晚睡的秘密被发现，😄棒棒哒

# ES6 Promise

## 同步與非同步的觀念

```javascript
function doWork() {
  var auntie = "漂亮阿姨";
  (function () {
    console.log('1. 打給阿姨')
    setTimeout(function () {
      console.log(auntie + '2. 回電');
      (function () {
        console.log('3. 洗碗');
      })();
    }, 0)
  })();
  (function () {
    console.log('3. 洗碗');
  })();
}
doWork();   // 執行
```

Promise 就是來解決巢狀問題。

## 製作 Promise 函式

```javascript
var runPromise = function(someone, timer, success) {
  console.log(someone + '開始奔跑');
  // Promise 從這開始
  return new Promise(function(resolve, reject) {
    if (success) {
      setTimeout(function() {
        resolve(someone + ' ' + (timer/1000) +'秒 抵達終點');
      }, timer);
    } else {
      reject(new Error(someone + '失敗'));
    }
  })
}

runPromise('小明', 2000, true)
.then(function (response) {
  console.log(response);
  return runPromise('胖虎', 3000, false);
})
.then(function (response) {
  console.log(response);
})
.catch(function (response) {
  console.log(response);
});
```

## Promise all

```javascript
Promise.all([runPromise('小明', 2000, true), runPromise('胖虎', 1000, true)])
.then(function (response) {
  console.log(response);
})
.catch(function (err) {
  console.log(err);
})
```

## 定義屬於自己的 Promise 函式

函式

- new Promise
    - resolve: 成功
    - reject: 失敗

使用 Promise 函式

- then: 接收成功的訊息
- catch: 接收失敗的訊息

```javascript
var runPromise = function(someone, timer, success) {
  console.log(someone + '開始奔跑');
  return new Promise(function(resolve, reject) {
    if (success) {
      setTimeout(function() {
        resolve(someone + ' ' + (timer/1000) + '秒 抵達終點');
      }, timer);
    } else {
      reject(new Error(someone + '失敗'));
    }
  });
};

runPromise('小明', 2000, true)
.then(function (response) {
  console.log(response);
  return runPromise('胖虎', 3000, false);
})
.then(function (response) {
  console.log(response);
})
.catch(function (response) {
  console.log(response);
});
```

## Promise All 與 Promise Race

```javascript
Promise.all([runPromise('小明', 2000, true), runPromise('胖虎', 3000, true)])
.then(function (response) {
  console.log(response[0], response[1]);
})
.then(function (err) {
  console.log(err);
})
```

```javascript
Promise.race([runPromise('小明', 2000, true), runPromise('胖虎', 1000, true)])
.then(function (response) {
  console.log(response); // 只會接收比較快的，胖虎
})
.then(function (err) {
  console.log(err);
})
```

## Firebase 與 Promise

```javascript
var database = firebase.database();

var categoriesPath = '/categories/';
var categoriesRef = database.ref(categoriesPath);
var articlesPath = '/articles/';
var articlesRef = database.ref(articlesPath);

articlesRef.child('-L1zre5jC4r4hNj9MENG').once('value').then(function (snapshot) {
  console.log(snapshot.val());
  var categoryId = snapshot.val().category;
  console.log(categoryId);
  categoriesRef.child(categoryId).once('value').then(function (snapshot) {
    console.log(snapshot.val());
  });
});

// 以上會不斷巢狀下去，優化寫法：

articlesRef.child('-L1zre5jC4r4hNj9MENG').once('value').then(function (snapshot) {
  console.log(snapshot.val());
  var categoryId = snapshot.val().category;
  console.log(categoryId);
  return categoriesRef.child(categoryId).once('value');
})
.then(function (snapshot) {
  console.log(snapshot.val());
});;
```
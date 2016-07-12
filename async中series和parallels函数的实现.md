# async中series和parallels函数的实现
## series
```javascript
function series(tasks, callback) {
     var results = [];
     var next = function() {
          var task = tasks.shift();
          if (task) {
               task(function(error, result) {
                    if (error) {
                         callback(error, results);
                    } else {
                         results.push(result);
                         next();
                    }
               })
          } else {
               callback(null, results);
          }
     }
     next();
}
```
## parallels
```javascript
function parallels(tasks, callback) {
     var results = [];
     var count = 0;
     var result_count = 0;
     var next = function() {
          var task = tasks[count];
          var index = count;
          if (task) {
               count++;
               task(function(error, result) {
                    if (error) {
                         callback(error, results);
                    } else {
                         results[index] = result;
                         if (++result_count == tasks.length) {
                              callback(null, results);
                         }
                    }
               })
               next();
          }

     }
     next();
}
// 测试
parallels([

     function(cb) {
          setTimeout(function() {
               console.log('task done: 1');
               cb(null, 1);
          }, 200);
     },
     function(cb) {
          setTimeout(function() {
               console.log('task done: 1');
               cb(null, 2);
          }, 200);
     },
     function(cb) {
          setTimeout(function() {
               console.log('task done: 1');
               cb(null, 3);
          }, 200);
     }
], function(error, results) {
     if (!error) {
          console.log(results);
     } else {
          console.log('Tasks failed.')
     }
})
```
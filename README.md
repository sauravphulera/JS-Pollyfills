# JS-Pollyfills
    Polyfills


## 1. Map

```
Array.prototype.myMap = function(callback) {
    let result = [];
    for (let i = 0; i < this.length; i++) {
        // Ensure callback is called with the correct arguments
        result.push(callback(this[i], i, this));
    }
    return result;
};

// Example usage
const numbers = [1, 2, 3];
const doubled = numbers.myMap(num => num * 2);
console.log(doubled); // Output: [2, 4, 6]
```

## 2.Filter

```
Array.prototype.myFilter = function(callback) {
    let result = [];
    for (let i = 0; i < this.length; i++) {
        if (callback(this[i], i, this)) {
            result.push(this[i]);
        }
    }
    return result;
};
```

## 3. Reduce

```
Array.prototype.myReduce = function(callback, initialValue) {
    let accumulator = initialValue === undefined ? this[0] : initialValue;
    let startIndex = initialValue === undefined ? 1 : 0;

    for (let i = startIndex; i < this.length; i++) {
        accumulator = callback(accumulator, this[i], i, this);
    }
    return accumulator;
};
```



## 4. For Each


```
Array.prototype.myForEach = function(callback) {
    for (let i = 0; i < this.length; i++) {
        callback(this[i], i, this);
    }
};

// Example usage
const numbers = [1, 2, 3];
numbers.myForEach(num => console.log(num));
// Output:
// 1
// 2
// 3
```



## 5. Call, Apply and Bind
Call

```
Function.prototype.myCall = function (context,...args) {
		if(typeof this !== 'function') {
    	  throw new TypeError('myCall must be called on a function');
        return;
    }
    const fn = Symbol();
    context[fn] = this;
    const result = context[fn](...args);
    delete context[fn];
    
    return result;
}
```


## b.Apply
```
Function.prototype.myApply = function (context, args) {
    if (typeof this !== 'function') {
        throw new TypeError('myApply must be called on a function');
    }

    context = context || globalThis;
    const fnSymbol = Symbol();
    context[fnSymbol] = this;

    const result = context[fnSymbol](...(args || [])); 
    delete context[fnSymbol];

    return result;
};
```




## C. Bind
```		
Function.prototype.myBind = function (context, ...bindArgs) {
    if (typeof this !== 'function') {
        throw new TypeError('myBind must be called on a function');
    }

    const originalFunc = this;

    return function (...callArgs) {
        return originalFunc.apply(context, [...bindArgs, ...callArgs]);
    };
};
```


// Example usage:
const greetJohn = greet.myBind(person, 'Hey');
console.log(greetJohn('.')); // Hey, John.





## Async.series and Async.parallel
```
const asyncUtils = {
    // Execute tasks in parallel
    parallel: function(tasks, finalCallback) {
        let results = [];
        let completedTasks = 0;
        let hasErrorOccurred = false;

        tasks.forEach((task, index) => {
            task((err, result) => {
                if (hasErrorOccurred) return; // Skip if error already occurred

                if (err) {
                    hasErrorOccurred = true;
                    return finalCallback(err);
                }

                results[index] = result;
                completedTasks++;

                // Call the finalCallback when all tasks are completed
                if (completedTasks === tasks.length) {
                    finalCallback(null, results);
                }
            });
        });
    },

    // Execute tasks in series
    series: function(tasks, finalCallback) {
        let results = [];
        let currentTaskIndex = 0;

        function executeNextTask() {
            if (currentTaskIndex === tasks.length) {
                return finalCallback(null, results); // All tasks completed
            }

            const task = tasks[currentTaskIndex];
            task((err, result) => {
                if (err) {
                    return finalCallback(err); // Stop execution if error occurs
                }

                results.push(result);
                currentTaskIndex++;
                executeNextTask(); // Execute the next task in the series
            });
        }

        executeNextTask(); // Start the first task
    }
};

// Example async tasks
function task1(callback) {
    setTimeout(() => {
        console.log("Task 1 completed");
        callback(null, "Result 1");
    }, 1000);
}

function task2(callback) {
    setTimeout(() => {
        console.log("Task 2 completed");
        callback(null, "Result 2");
    }, 500);
}

function task3(callback) {
    setTimeout(() => {
        console.log("Task 3 completed");
        callback(null, "Result 3");
    }, 700);
}

// Usage Example

// 1. Async Parallel Execution
console.log("Running tasks in parallel:");
asyncUtils.parallel([task1, task2, task3], (err, results) => {
    if (err) {
        console.error("Error: ", err);
    } else {
        console.log("All tasks completed in parallel. Results: ", results);
    }
});

// 2. Async Series Execution
console.log("\nRunning tasks in series:");
asyncUtils.series([task1, task2, task3], (err, results) => {
    if (err) {
        console.error("Error: ", err);
    } else {
        console.log("All tasks completed in series. Results: ", results);
    }
});
```



## Deep Clone an Object

```
function deepClone(obj) {
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }

    // Create an array or object to hold the values
    const copy = Array.isArray(obj) ? [] : {};

    // Recursively copy each property
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            copy[key] = deepClone(obj[key]);
        }
    }
    return copy;
}
```



## Debounce
```
function debounce(callback, timer) {
	let timeout;
  
  return (...args) => {
  	clearTimeout(timeout);
    timeout = setTimeout(() => {
     callback(...args);
    }, timer)
  }
}
const debouncedSearchInput = debounce(handleSearchInput, 300);
```



## Throttle
```
function throttle(func, wait) {
 // your code here
 let timeout, shouldCallFn = true, lastArgs = null;


 return function throttled(...args) {
   // console.log(func)


   if(shouldCallFn) {
     func.apply(this, args);
       shouldCallFn = false;
       if(timeout) {
         clearTimeout(timeout);
       }
       timeout = () => setTimeout(() => {
         shouldCallFn = true;
         if(lastArgs) {
           func.apply(this, lastArgs);
           shouldCallFn = false;
           lastArgs = null;
           timeout();
         }
       }, wait)


       timeout();
   } else {
     lastArgs = [...args];
   }
   return;


 }
}
```





## Throttle With Leading and Trailing options
```
function throttle(func, wait, option = {leading: true, trailing: true}) {
// your code here
let timeout, shouldCallFn = true, lastArgs = null;
return function throttled(...args) {
   // check to avoid creating extra timeouts
  if(shouldCallFn) {
   shouldCallFn = false;
   // for leading true immediately call func else stash
   if(option.leading) {
     func.apply(this, args);
   } else {
     lastArgs = [...args];
   }
      if(timeout) {
        clearTimeout(timeout);
      }
      timeout = () => setTimeout(() => {
        shouldCallFn = true;
        if(lastArgs && option.trailing) {
	    // cal the last func if trailing is true
          func.apply(this, lastArgs);
          shouldCallFn = false;
          lastArgs = null;
          timeout();
        }
      }, wait)
      timeout();

    
  } else {
    lastArgs = [...args];
  }
}
}
```



## Promise Object Implementation

```
class PromiseSimple {
		constructor(executionFunction) {
     		  this.handleError = () => {};
              this.PromiseChain = [];
              this.resolve = this.resolve.bind(this);
              this.reject = this.reject.bind(this);
    		  executionFunction(this.resolve, this.reject);

           }
    
    then(callback) {
    		this.PromiseChain.push(callback);
    		return this;
    }
    catch(callback) {
    		this.handleError = callback;
    		return this;
    }
    
    resolve(res) {
    		try {
        	this.PromiseChain.forEach((cb) => {
          		cb(res);
          })
        } catch(err) {
          this.PromiseChain = [];

          this.reject(err);
        }
    }
    
    reject(err) {
    		this.handleError(err);
    }
}
```




## Publisher Subscriber
```
class PubSub {
  constructor() {
    this.subscribers = {};
  }

  subscribe(eventName, callback) {
    // Subscribe a callback to an eventName
    if(!this.subscribers[eventName])
    	this.subscribers[eventName]= [callback];
    else
    	this.subscribers[eventName].push(callback);
  }

  publish(eventName, data) {
    // Execute callbacks for an eventName
    if(this.subscribers[eventName]) {
    	for(let subs of this.subscribers[eventName]) {
      	subs(data);
      }
    }
    
  }

	unsubscribe(eventName, callback) {
    // Remove a previously added callback
    if(this.subscribers[eventName]) {
    		let i = this.subscribers[eventName].indexOf(callback);
        this.subscribers[eventName].splice(i, 1);
    }
  } 
}
```


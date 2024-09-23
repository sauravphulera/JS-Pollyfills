# JS-Pollyfills
    Polyfills


## 1. Map

```javascript
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

```javascript
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

```javascript
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


```javascript
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

```javascript
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
```javascript
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
```javascript		
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
```javascript
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

```javascript
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
```javascript
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

## Debounce with leading and trailing option
```javascript
function debounce(func, wait, option = {leading: false, trailing: true}) {
  // your code here
  let timeout, cooling = false, lastFun = true;

  return function (...args) { 
    if(timeout) {
      clearTimeout(timeout);
    }
    if(option.leading && !cooling) {
      func.apply(this, args);

      // to make sure same func is not called again if trailing is also true
      lastFun = false;
      cooling = true;
      
    } else {
      lastFun = true;
    }

    timeout = setTimeout(() => {
      if(option.trailing &&  lastFun) {
        func.apply(this, args);
      }
      cooling = false;
    }, wait)

  }
}
```



## Throttle
```javascript
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
```javascript
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

```javascript
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

## Promise.all 

```javascript
// Polyfill for Promise.all
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    let results = [];
    let completedPromises = 0;

    // Check if the input is empty and resolve with an empty array
    if (promises.length === 0) {
      resolve([]);
    }

    promises.forEach((promise, index) => {
      // Convert the value to a promise using Promise.resolve
      Promise.resolve(promise)
        .then((value) => {
          // Store the resolved value in results array
          results[index] = value;
          completedPromises++;

          // If all promises have resolved, resolve the main promise with results array
          if (completedPromises === promises.length) {
            resolve(results);
          }
        })
        .catch((error) => {
          // If any promise rejects, reject the main promise with the error
          reject(error);
        });
    });
  });
}
```

## Promise.all with concurrency
```javascript
function promiseAllWithConcurrency(promises, concurrency) {
  return new Promise((resolve, reject) => {
    let results = [];
    let runningCount = 0; // Number of currently running promises
    let currentIndex = 0;  // The index of the next promise to run
    let completedPromises = 0;

    // Start the next promise in the queue
    const runNext = () => {
      if (completedPromises === promises.length) {
        return resolve(results);
      }

      // If no more promises are left to run, stop the process
      if (currentIndex >= promises.length) {
        return;
      }

      const index = currentIndex; // Capture the current promise's index
      const promise = promises[currentIndex]; // Get the next promise
      currentIndex++; // Move the index to the next promise

      runningCount++;

      // Convert to a promise to handle non-promise values
      Promise.resolve(promise)
        .then((value) => {
          results[index] = value;
          completedPromises++;
          runningCount--;

          // When a promise finishes, start the next one
          if (completedPromises === promises.length) {
            resolve(results); // Resolve when all promises are done
          } else {
            runNext(); // Start a new promise
          }
        })
        .catch((error) => {
          reject(error); // Reject as soon as one promise fails
        });

      // Ensure that we maintain the concurrency limit
      if (runningCount < concurrency) {
        runNext();
      }
    };

    // Start the initial promises (up to the concurrency limit)
    for (let i = 0; i < Math.min(concurrency, promises.length); i++) {
      runNext();
    }
  });
}
```
### Explanation:
1. Concurrency control: The function runs at most concurrency number of promises simultaneously. Once a promise finishes (either resolves or rejects), the next promise from the queue starts.
2. Run-next mechanism: After a promise completes, runNext is called to continue running the next pending promise in the list.
3. Edge case: If the input array is empty, the function immediately resolves with an empty array.



## Publisher Subscriber
```javascript
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


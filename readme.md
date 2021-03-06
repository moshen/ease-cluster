# Ease-Cluster

Dead-simple one-liner for clustered Node.js apps.

This project is a fork of [Throng](https://github.com/hunterloftis/throng). 

Ease-Cluster differs from Throng in that it will wait for the master function to finish
before forking off child processes. An important feature if you're wanting to initialize
globals asynchronously before forking.

Runs X workers and respawns them if they go down.
Correctly handles signals from the OS.

```js
const easeCluster = require('ease-cluster');

easerCluster((id) => {
  console.log(`Started worker ${id}`);
});
```

```
$ node example
Started worker 1
Started worker 2
Started worker 3
Started worker 4
```

## Installation

```
npm install --save ease-cluster 
```


## Use

```js
easeCluster(startFunction);
```
Simplest; fork 1 worker per CPU core.

```js
easeCluster(3, startFunction);
```
Specify a number of workers.

```js
easeCluster({
  workers: 16,
  grace: 1000,
  master: masterFunction,
  start: startFunction
});
```
More options.

```js
easeCluster((id) => {
  console.log(`Started worker ${id}`);

  process.on('SIGTERM', function() {
    console.log(`Worker ${id} exiting`);
    console.log('Cleanup here');
    process.exit();
  });
});
```
Handling signals (for cleanup on a kill signal, for instance).

## All Options (with defaults)

```js
easeCluster({
  workers: 4,       // Number of workers (cpu count)

  lifetime: 10000,  // ms to keep cluster alive (Infinity)

  grace: 4000,      // ms grace period after worker SIGTERM (5000)

  master: masterFn, // Function to call when starting the master process
                    // master is a Promise and will resolve before `start`
                    // function is called. Removing the possibility of a race
                    // condition.

  start: startFn,   // Function to call when starting the worker processes

  env: {            // Additional env vars to place on worker processes
    myenv1:'envVal',// that don't already exist on the master process
    myenv2:'envVal2'
  } 
});
```

## A Complex example

```js
const easeCluster = require('ease-cluster');

easeCluster({
  workers: 4,
  master: startMaster,
  start: startWorker,
  env: {
    NODE_ENV: 'dev',
    MY_OTHER_ENV_VAR: 17,
    THIS_VAL_TOO: 'value'
  }
});

// This will only be called once
function startMaster() {
  console.log('Started master');
}

// This will be called four times
function startWorker(id) {
  console.log(`Started worker ${ id }`);

  process.on('SIGTERM', () => {
    console.log(`Worker ${ id } exiting...`);
    console.log('(cleanup would happen here)');
    process.exit();
  });
}
```

```
$ node example-complex.js
Started master
Started worker 1
Started worker 2
Started worker 3
Started worker 4

$ killall node

Worker 3 exiting...
Worker 4 exiting...
(cleanup would happen here)
(cleanup would happen here)
Worker 2 exiting...
(cleanup would happen here)
Worker 1 exiting...
(cleanup would happen here)
```

## Tests

```
npm test
```

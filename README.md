> **`piri-piri`** is a browser orchestration to enable decentralized browser applications tests. Ah and it is hot :)

![](/img/logo.png)


**important disclaimer -** This tool is recent, it has a its rough edges. I will keep devoting time to it because I have a necessity for a tool like this. If you have happen to have a scenario where it would be useful, please share, I would love to hear and help make it happen!

# Badgers
[![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/diasdavid/piri-piri?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![](https://cldup.com/pgZbzoshyV-3000x3000.png)](http://www.gsd.inesc-id.pt/)

# Why does it exist

There are a panoply of excellent browser testing frameworks and services available today, however their focus is on testing browser implementations (CSS, HTML and JavaScript) and user interactions of the apps their are testing (clicks, mouse movements, what the user sees). 

When it comes to testing to test a decentralized browser app or library, the focus stops being how a browser implements a specific behaviour, but how the decentralized network handles node joins and leaves and if nodes are effectively communicating between each other. In this scenario, we have several events that the server never sees or that the server never instructs the clients to do, so we need to create a new way to coordinate the browser joins and leaves and also how they interact between each other remotely and this is were `piri-piri` comes into play.


The specific set of problems `piri-piri` tries to solve:

- browser times X, where 1<=X<=virtually unlimited - Most browser testing frameworks only let you launch a couple of browsers at a time, `piri-piri` aims to launch several browsers and/or tabs to load a webpage, in a local or distributed fashion.

- instruct browsers on demand - Since there is a ton of stuff happening on browser decentralized apps, we can't just write a script to test and listen to events that happens in a single browser, there are triggers coming from all of them.

- gather information and evaluate the state as a whole - collect the events and data generated by each browser and assess if the order was correct with pseudo external consistency

**why `piri-piri`? Well, to be honest, since I got to learn about SauceLabs in 2012 (during LXJS over some Nachos and Tabasco Hot Sauce), browser testing for me was always connected to spicy and sauce, so that inspired me to pick the one that is very famous on the portuguese cousine, that is, `piri-piri` :)


# How to use it (API)

```
var pp = require('piri-piri');
```

## Launching browsers 

Open a chrome browser tab in a given url
```
pp.farm.spawn('http://www.theinternet.com', 'chrome', function (err) {
  if (err) { 
    console.log(err);
  }
});
```

Close the browser process (all tabs)
```
pp.farm.stop(function (err){
  if (err) { 
    console.log(err);
  }
});
```

## How to test a browser module

Install `piri-piri` and `piri-piri.client`, the first will be our server side component that will orquestrate the several browsers that are launched, the later is a simple tool to hook it to the server

```
$ npm i piri-piri piri-piri.client
```

### Instruct

Create a file that is going to be used to require the module you want to test and instruct it with [`piri-piri.client`](https://github.com/diasdavid/piri-piri.client)


```
$ touch serve-this.js
$ vim serve-this.js
```

```
var pp = require('piri-piri.client');

window.app = {
  init: function () {

    // by default, piri-piri serves on port 9876
    var options = { url: 'http://localhost:9876' };
    
    pp.start(options, function () {
      console.log('piri-piri client is ready');

      // register a simple action for piri-piri to invoke on the client
      pp.register('sum', function (data) {
        var total = data.a + data.b;
      });

      // register an action for piri-piri to invoke on the client that will return a value to the piri-piri server, so it can be later evaluated
      pp.register('sum-return', function (data) {
        pp.tell({total: data.a + data.b});
      });

    });
  }
};

window.app.init();
```

`piri-piri` uses `browserify` inside `moonboots_hapi` to serve your module

### Serve

Create a `.js` file to be the starting point of our piri-piri tests

```
$ touch test.js
$ vim test.js
```

```
var pp = require('piri-piri');

var options = {
  path: __dirname + '/serve_this.js',
  port: 9876,
  host: 'localhost'
};

pp.start(options, function(err) {
  if (err) { 
    console.log(err); 
  }
});
```

### Spawn a browser

Spawn a browser and wait for it to get connected

```
pp.farm.spawn(pp.uri(), 'chrome');
pp.waitForClients(1, function() {
      
});    
```

### Execute

```
var clientIds = pp.manager.getClientIDs();
var clientA = pp.manager.getClient(clientIds[0]);
clientA.command('sum', { a:1, b:8  });
```


### Execute and Evaluate

```
clientA.command('sum-return', { a:1, b:8 });

clientA.waitToReceive(1, function () {
  var queue = clientA.getQ();
  console.log('Sum Result was: ', queue[0].data.total);
  clientA.clearQ();     
});

```


# How to run the tests

**Make sure you have Google Chrome installed**

```
$ git clone https://github.com/diasdavid/piri-piri.git
$ git clone https://github.com/diasdavid/piri-piri.client.git
$ cd piri-piri
$ npm i 
$ npm test
```


``

# Roadmap

## piri-piri
- [X] serve the app (browserify main .js file and serve that)
- [X] save clients as they connect
- [X] queue messages by client
- [ ] filters (like packet sniffers)
- [ ] order all the messages using pseudo external consistency
- [X] launch browser tabs on demand
- [~] **priority** launch browser tabs properly and scalably (see "Spawning more browsers" Issue below)

## piri-piri.client
- [X] connect to a piri-piri host
- [X] register actions to be executed when host commands
- [X] send messages to the host
- [X] encapsulate the message to host with more useful information

# Current Issues ( please contribute :) )

These are implementation issues I've faced and that I would be more than thankful to hear your input. 

https://github.com/diasdavid/piri-piri/issues

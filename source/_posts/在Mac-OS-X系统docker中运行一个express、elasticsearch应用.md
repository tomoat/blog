---
title: 在Mac OS X系统docker中运行一个express、elasticsearch应用
**RUNNING A NODE.JS EXPRESS APP WITH AN ELASTICSEARCH BACK END ON DOCKER ON MAC OS X**
---

I have been playing around with Docker for some days now and like to share what I have learned so far. Therefore this article will describe, as of March 31st, how to set up a productive development environment with Docker on Mac OS X and how to run a Node.js Express app and Elasticsearch in two Docker containers. The app itself will be a pretty simple fuzzy auto-suggest for book titles which are pre-loaded into the Elasticsearch container.

You can find this article's [code on GitHub](https://github.com/andre-hh/nodejs-express-elasticsearch-docker).

## INSTALLING DOCKER ON MAC OS X

First of all, you have to install [Docker Toolbox](https://www.docker.com/products/docker-toolbox). This will set up a virtual machine (VM) named "default" running Linux inside [VirtualBox](https://www.virtualbox.org/). All Docker containers will reside within this VM, as it is not possible to run Docker containers on Mac OS X directly. The result are three layers: Max OS X itself, the VM and the containers inside the VM. You must keep these layers in mind as each layer has its own file system, users and groups.

Docker Toolbox maps your Mac OS X "/Users" folder into the VM, where it is owned by a user named "docker" with uid 1000. This implies two things:

1. When you share code from your Mac this code must reside with your /Users directory.
2. When you access files within the /Users share from your Docker container, you should do so with a user with uid 1000 to avoid permission errors.

Now its time to start your VM via docker-machine (which comes with the Docker Toolbox):

```
$ docker-machine start default
```

Afterwards you must add some environment variables:

```
$ eval "$(docker-machine env default)"
```

Find our where your VM is running:

```
$ docker-machine url
tcp://192.168.99.100:2376
```

As this command binds the environment variables only to the current shell you have to execute it again whenever you open an new one. [Adding them to your shell permanently](http://unix.stackexchange.com/questions/117467/how-to-permanently-set-environmental-variables)avoids this pain.

You can find a little more detailed read about this basic Docker setup on Mac OS X [here](https://medium.com/@brentkearney/docker-on-mac-os-x-9793ac024e94#.uxxlq712k).

## CREATE THE PROJECT STRUCTURE

After having installed Docker Toolbox its time to create the project structure:

```
$ tree
.
├── server
└── docker-compose.yml
```

We'll place our Node.js Express app inside the `/server` directory.

As we have to work with two containers - our Node.js Express server and our Elasticsearch persistence - we use [Docker Compose](https://docs.docker.com/compose/) for defining and running multi-container Docker applications.

Remember, your project files must reside within your /Users directory so that they are available in the VM and can be used within the containers.

## STARTING ELASTICSEARCH AS A DOCKER CONTAINER

To get a first Docker container running, modify your `docker-compose.yml`:

```
elasticsearch:
  image: elasticsearch
  ports:
    - '9200:9200'
```

Build and run the Elasticsearch container:

```
$ docker-compose up
```

If everything worked fine, you should have a running Elasticsearch container which is based on [Docker Hub's Elasticsearch image](https://hub.docker.com/_/elasticsearch/). We don't need to map the host's port 9200 to the container's port 9200 later on (as we'll simply link containers), but its good to check whether everything works:

```
$ curl http://192.168.99.100:9200 
{
  "name" : "Dracula",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.2.1",
    "build_hash" : "d045fc29d1932bce18b2e65ab8b297fbf6cd41a1",
    "build_timestamp" : "2016-03-09T09:38:54Z",
    "build_snapshot" : false,
    "lucene_version" : "5.4.1"
  },
  "tagline" : "You Know, for Search"
}
```

## RUN A NODE.JS EXPRESS APP ON DOCKER

Next, we'll set up our Node.js app which will utilize our Elasticsearch container. Replace the contents of your `docker-compose.yml` file:

```
server:
  build: ./server
  command: sh /home/app/start.sh
  ports:
    - '5000:5000'
  volumes:
    - /Users/andre/IdeaProjects/nodejs-express-elasticsearch-docker/server:/home/app
  links:
    - elasticsearch

elasticsearch:
  image: elasticsearch
```

This creates an additional container named "server" that is linked to the "elasticsearch" container. We map our local project directory into the container at `/home/app` to share project files. As we have done previously with Elasticsearch, we map the host's port 5000 to the container's port 5000 (where our Node.js app will be running) for testing purposes. Running `docker-compose up` (which won't work yet) will build the "server" container according to the contents of our `Dockerfile`, which we'll create in `/server`:

```
FROM node:4.3.2

RUN useradd --user-group --create-home --shell /bin/false app

RUN npm install nodemon -g

ENV HOME=/home/app

USER root
RUN chown -R app:app $HOME
USER app
```

We use node:4.3.2 as our base image and add a user named app because we shouldn't work as root. [nodemom](http://nodemon.io/) will monitor our source files for any changes and automatically restart the server, which is perfect for development (to keep things simple, this article does not differentiate between development and production environments).

The entrypoint of the container is `sh /home/app/start.sh` which has been defined in the `docker-compose.yml` file. Create `start.sh` with the following contents in your `/server`directory:

```
#!/bin/bash

cd /home/app
npm install

while ! curl http://elasticsearch:9200; do sleep 1; done;

nodemon -L /home/app
```

This script installs the dependencies defined in `package.json` within the container and **not**on our host (they are only shared with the host due to the shared volume). After having installed the dependencies we wait for the Elasticsearch container to be up and running before we start our Node.js Express app with nodemon (we need to start nodemon with the -L option for "legacy watch" as our application won't restart in this environment otherwise).

Create `package.json` in the `/server` directory:

```
{
  "name": "server",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "elasticsearch": "10.1.3",
    "body-parser": "^1.12.4",
    "express": "^4.12.0"
  }
} 
```

Next, we'll create a little Express.js app that connects to Elasticsearch based on Raanan Weber's article " [Getting started with Elasticsearch and Express.js](https://blog.raananweber.com/2015/11/24/simple-autocomplete-with-elasticsearch-and-node-js/)". Let's get started with an elasticsearch module in `/server/elasticsearch.js`:

```
var elasticsearch = require('elasticsearch');

var elasticClient = new elasticsearch.Client({
    host: 'elasticsearch:9200',
    log: 'info'
});

var indexName = 'books';

function indexExists() {
    return elasticClient.indices.exists({
        index: indexName
    });
}
exports.indexExists = indexExists;

function initIndex() {
    return elasticClient.indices.create({
        index: indexName
    });
}
exports.initIndex = initIndex;

function deleteIndex() {
    return elasticClient.indices.delete({
        index: indexName
    });
}
exports.deleteIndex = deleteIndex;

function initMapping() {
    return elasticClient.indices.putMapping({
        index: indexName,
        type: 'book',
        body: {
            properties: {
                title: { type: 'string' },
                suggest: {
                    type: 'completion',
                    analyzer: 'simple',
                    search_analyzer: 'simple',
                    payloads: true
                }
            }
        }
    });
}
exports.initMapping = initMapping;

function addBook(book) {
    return elasticClient.index({
        index: indexName,
        type: 'book',
        body: {
            title: book.title,
            suggest: {
                input: book.title.split(' '),
                output: book.title,
                payload: book.metadata || {}
            }
        }
    });
}
exports.addBook = addBook;

function getSuggestions(input) {
    return elasticClient.suggest({
        index: indexName,
        type: 'book',
        body: {
            docsuggest: {
                text: input,
                completion: {
                    field: 'suggest',
                    fuzzy: true
                }
            }
        }
    })
}
exports.getSuggestions = getSuggestions;
```

I guess most of this file is pretty self-explanatory; it connects to the Elasticsearch container (we use the name defined in `docker-compose.yml` here) and provide various methods to work with Elasticsearch.

Next, we'll start our Express.js app and fill Elasticsearch' index with some book titles by creating `index.js` in our `/server` directory:

```js
var express = require('express');
var app = express();
var bodyParser = require('body-parser');

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

app.use('/books', require('./routes/books'));

var elastic = require('./elasticsearch');
elastic.indexExists().then(function (exists) {
    if (exists) {
        return elastic.deleteIndex();
    }
}).then(function () {
    return elastic.initIndex().then(elastic.initMapping).then(function () {
        var promises = [
            'The Lord of the Rings',
            'The Hobbit',
            'The Little Prince',
            'Harry Potter and the Philosopher`s Stone',
            'And Then There Were None'
        ].map(function (bookTitle) {
            return elastic.addBook({
                title: bookTitle,
                content: bookTitle + " content",
                metadata: {
                    titleLength: bookTitle.length
                }
            });
        });
        return Promise.all(promises);
    });
});

// Error handling middleware must be after all other middleware and routing.
// Handle error in development mode.
if (app.get('env') === 'development') {
    console.log('running in dev mode');
    app.use(function (err, req, res, next) {
        res.status(500).json(err.stack);
    });

// Handle error in production mode.
} else {
    console.log('running in production mode');
    app.use(function (err, req, res, next) {
        res.status(500).json(err.message);
    });
}

app.listen(5000, function () {
    console.log('Listening server on port 5000');
});
```

Finally, we define the one and only route of our Express.js app in `/server/routes/books.js`:

```js
var express = require('express');
var router = express.Router();

var elastic = require('../elasticsearch');

router.get('/suggest/:input', function (req, res, next) {
    elastic.getSuggestions(req.params.input).then(function (result) {
        res.json(result)
    });
});

module.exports = router;
```

After having executed `docker-compose up` we should be able to utilize Elasticsearch' fuzzy auto-complete feature:

```bash
$ curl http://192.168.99.100:5000/books/suggest/hobbet
{
    _shards: {
        total: 5,
        successful: 5,
        failed: 0
    },
    docsuggest: [
        {
            text: "hobbet",
            offset: 0,
            length: 6,
            options: [
                {
                    text: "The Hobbit",
                    score: 1,
                    payload: {
                        titleLength: 10
                    }
                }
            ]
        }
    ]
}
```

And that's it!

Again, the whole code is available [on GitHub](https://github.com/andre-hh/nodejs-express-elasticsearch-docker), you can play along with it as much as you want.

Please share your feedback in the comments below and, if you liked this post, follow me on[Twitter](https://twitter.com/AndreKolell) and [github](https://github.com/andre-hh).



[Running a Node.js Express app with an Elasticsearch back end on Docker on Mac OS X | André Kolell](http://www.andrekolell.de/blog/running-nodejs-express-app-with-elasticsearch-on-docker-on-mac)
### MEAN STACK DEPLOYMENT

#### STEP-1
#### Install node.js

node.js is a javascript runtime built on chrme's V8 javascript engine. node.js is used here to setup express routes and angular controllers.

##### update ubuntu
    sudo apt update

#### upgrade ubuntu

    sudp apt upgrade

#### Add certificates

    sudo apt -y install curl dirmngr apt-transport-https lsb-release cacertificates
    curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -

#### install node.js

    sudo apt install -y nodejs

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-05%20145735.png)



#### INSTALL mongoDB
mondodb stores data in flexible json like documents.Fields in a database can vary from document to document
and data structure can be changed over time.For our example we are adding book records to Mongodb that contain booknames,
isbnnumber,author and number of pages.

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv
    0C49F3730359A14518585931BC711F9BA15703C6

    sudo apt install gnupg2

    sudo gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0C49F3730359A14518585931BC711F9BA15703C6


    echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list


Kindly use the commands below to install ‘mongodb’

Add gpg keys:

    curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

Add mongodb repo:
    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

Update package manager:
    sudo apt update

Install mongodb:
    sudo apt install -y mongodb-org

Verify installation:
    mongod --version

#### Issue encountered

mongodb failed to start.

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-07%20113053.png)


#### solution

Create a new service file for MongoDB.

    sudo nano /etc/systemd/system/mongodb.service

Add the following content to the file:

    [Unit]
    Description=MongoDB Database Server
    Documentation=https://docs.mongodb.org/manual
    After=network.target

    [Service]
    User=mongodb
    ExecStart=/usr/bin/mongod --config /etc/mongod.conf
    PIDFile=/var/run/mongodb/mongod.pid
    LimitNOFILE=64000
    TimeoutStopSec=60
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-07%20114917.png)


Save and close the file (Ctrl+O, Enter, Ctrl+X for nano).

Reload the systemd daemon:

    sudo systemctl daemon-reload

Enable and start the MongoDB service:

    sudo systemctl enable mongodb
    sudo systemctl start mongodb

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-07%20115018.png)

Check the status of the MongoDB service:

    sudo systemctl status mongodb

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-07%20115146.png)

install [npm] nodepackage manager(https://www.npmjs.com)

    sudo apt install -y npm

install body-parser package
we need body parser to help us process JSON files passed in requests to the server.

    sudo npm install body-parser

create a folder named Books

    mkdir Books

move into the directory

    cd Books

in the books directory initialize npm project

    npm init

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-07%20120634.png)

add a file to it called server.js

    vi server.js

paste code into file

    var express = require('express');
    var bodyParser = require('body-parser');
    var app = express();
    app.use(express.static(__dirname + '/public'));
    app.use(bodyParser.json());
    require('./apps/routes')(app);
    app.set('port', 3300);
    app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
    });


#### Step3 install expressjs and setup routes to its server
Express is a minimal and flexible node.js web application that provides feature for web and mobile applications.
we will use express to pass information to and from our mongodatabase.
we will also use mongoose package which provides a straight-forward ,schema-based solution to model our application.
we will use mongoose to establish a schema for our database to store data for our book register

    sudo npm install express mongoose

in the books folder create a folder named apps  and cd into apps

    mkdir apps && cd apps

create a file named routes.js

    touch routes.js

open the routes file

    vi routes.js

paste code into routes.js

    var Book = require('./models/book');

    module.exports = function(app) {
        app.get('/book', function(req, res) {
            Book.find({}, function(err, result) {
                if (err) throw err;
                res.json(result);
            });
        });

        app.post('/book', function(req, res) {
            var book = new Book({
                name: req.body.name,
                isbn: req.body.isbn,
                author: req.body.author,
                pages: req.body.pages
            });

            book.save(function(err, result) {
                if (err) throw err;
                res.json({
                    message: "Successfully added book",
                    book: result
                });
            });
        });

        app.delete("/book/:isbn", function(req, res) {
            Book.findOneAndRemove(req.query, function(err, result) {
                if (err) throw err;
                res.json({
                    message: "Successfully deleted the book",
                    book: result
                });
            });
        });

        var path = require('path');
        app.get('*', function(req, res) {
            res.sendFile(path.join(__dirname + '/public', 'index.html'));
        });
    };


![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-08%20092012.png)

in the apps folder create a folder named models and cd into it

    mkdir models && cd models

create a file named books.js

    touch books.js

open file 

    vi books.js


paste code into it

    var mongoose = require('mongoose');

    // MongoDB connection
    var dbHost = 'mongodb://localhost:27017/test';
    mongoose.connect(dbHost);
    mongoose.connection;

    // Debugging
    mongoose.set('debug', true);

    // Define Book Schema
    var bookSchema = mongoose.Schema({
        name: String,
        isbn: { type: String, index: true },
        author: String,
        pages: Number
    });

    // Create Book model
    var Book = mongoose.model('Book', bookSchema);

    // Export Book model
    module.exports = Book;


#### step 4 access routes with angularjs

angular provides a web framework for creating dynamic views in a web application.we will use angular to connect our web page with 
express and perform actions on our book register.

change directory back to Books

    cd ../..

create a folder named public and cd into folder

    mkdir public && cd public

add a file named script.js and open it
    touch script.js
    vi script.js


paste code below into script.js file 

    var app = angular.module('myApp', []);

    app.controller('myCtrl', function($scope, $http) {
        $http({
            method: 'GET',
            url: '/book'
        }).then(function successCallback(response) {
            $scope.books = response.data;
        }, function errorCallback(response) {
            console.log('Error: ' + response);
        });

        $scope.del_book = function(book) {
            $http({
                method: 'DELETE',
                url: '/book/' + book.isbn
            }).then(function successCallback(response) {
                console.log(response);
            }, function errorCallback(response) {
                console.log('Error: ' + response);
            });
        };

        $scope.add_book = function() {
            var body = {
                "name": $scope.Name,
                "isbn": $scope.Isbn,
                "author": $scope.Author,
                "pages": $scope.Pages
            };

            $http({
                method: 'POST',
                url: '/book',
                data: body
            }).then(function successCallback(response) {
                console.log(response);
            }, function errorCallback(response) {
                console.log('Error: ' + response);
            });
        };
    });


![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-08%20093615.png)

in the public folder create a file called index.html

    touch index.html
    vi index.html

paste code below

    <!doctype html>
    <html ng-app="myApp" ng-controller="myCtrl">
    <head>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
        <script src="script.js"></script>
    </head>
    <body>
        <div>
            <table>
                <tr>
                    <td>Name:</td>
                    <td><input type="text" ng-model="Name"></td>
                </tr>
                <tr>
                    <td>Isbn:</td>
                    <td><input type="text" ng-model="Isbn"></td>
                </tr>
                <tr>
                    <td>Author:</td>
                    <td><input type="text" ng-model="Author"></td>
                </tr>
                <tr>
                    <td>Pages:</td>
                    <td><input type="number" ng-model="Pages"></td>
                </tr>
            </table>
            <button ng-click="add_book()">Add</button>
        </div>
        <hr>
        <div>
            <table>
                <tr>
                    <th>Name</th>
                    <th>Isbn</th>
                    <th>Author</th>
                    <th>Pages</th>
                    <th>Action</th>
                </tr>
                <tr ng-repeat="book in books">
                    <td>{{book.name}}</td>
                    <td>{{book.isbn}}</td>
                    <td>{{book.author}}</td>
                    <td>{{book.pages}}</td>
                    <td><button ng-click="del_book(book)">Delete</button></td>
                </tr>
            </table>
        </div>
    </body>
    </html>

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-08%20094035.png)

change directory back to books 

    cd ..

open port 3300 from aws security group to listen

start server by running command 

    node server.js

![alt text](https://github.com/techbeast911/mean_stack_deployment/blob/main/img/Screenshot%202024-06-08%20100434.png)


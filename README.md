# Real-Time Learning Tracker


The *Real-Time Learning Tracker* is an interactive learning widget that can be intergrated in a learner dashboard destined for MOOC learners on the  edX  platform.  This **real-time** feedback mechanism for edX learners consists of three different feedback complexity versions and each one of them uses low-level data from trace logs and converts it into behavioural indicators that describe several learning habits of the learners. By means of a bar chart appearing in a pop-up window at the bottom right of every MOOC page, it provides learners with real-time feedback on their learning behaviour comparing it to previously successful learners. 

The Complex version of the *Real-Time Learning Tracker* is presented in action via this screencast: https://www.youtube.com/watch?v=jlhfIt9iCAE&t=53s

## Functional Architecture

The working architecture of the *Real-Time Learning Tracker* has three components as shown in the figure below.

![alt text](https://github.com/gatou92/RealTimeLearningTracker/blob/master/images/RLT_FuncionalArchitecture.jpg)
 
1. **edX component (Javascript)**
	- contains the code responsible for real-time logging learners’ activity during the course.
	- contains the calculation of the metric values of each active learner in the course from 	the logged activity data 
	- loads each version (complex-intermediate-simple) of the *Real-Time Learning Tracker* on the edX pages in a pop-up window, based on learner’s edX id.

	The widget is plotted using [Highcharts](https://www.highcharts.com). The documentation for customizing the widget is [here](https://api.highcharts.com/highcharts/).

	Each version of the widget is included in the files [complex_version](https://github.com/gatou92/RealTimeLearningTracker/blob/master/edX_MOOC_pages/public/js/complex_version.js), [intermediate_version](https://github.com/gatou92/RealTimeLearningTracker/blob/master/edX_MOOC_pages/public/js/intermediate_version.js) and [simple_version](https://github.com/gatou92/RealTimeLearningTracker/blob/master/edX_MOOC_pages/public/js/simple_version.js) respectively.

2. **Server backend (Javascript)**
	- developed with the [node.js](https://nodejs.org/en/) server environment, the back-end is 
	the part of the *Real-Time Learning Tracker* responsible for managing requests.
	- each time a new event is traced from the *real-time logger* code, an ajax POST request is sent to the Node server which inturn logs the corresponding event to [MongoDB](https://www.mongodb.com) database.
	- on the other hand, during the widget generation step, an ajax GET request is sent to the server for acquiring the necessary logs for the learners’ metric computation.
	- in the same GET request also the already computed *Average* and *Most Engaged Graduate* profiles are retrieved

	The code for the server is included in the folder [Backend](https://github.com/gatou92/RealTimeLearningTracker/tree/master/Backend).

3. **Local component (Python)**
	- the successful learner profiles (**Average** and **Most Engaged Graduate**) are generated by calculating the metric values from low-level activity data form a previous edition of the edX course and are inserted into the MongoDB database.

	The code for the local, offline component is included in the folder [Offline_Computation](https://github.com/gatou92/RealTimeLearningTracker/tree/master/Offline_Computation). The current implementation calculates 7 metrics as presented below. 


## Metrics available

The metrics are calculated considering data generated from the first day of the course.

1. Number of video lecture watched
2. Number of quiz questions submitted
3. Amount of time spent on the course pages (in minutes)
4. Amount of time spent watching video lectures (in minutes)
5. The ratio of time spent watching video lectures while on the course pages (%)
6. The amount of forum contributions (#threads/#responses/#comments created) on the course pages
7. The ratio of learners' Engagement (%) - aggregated activity during the course computed based on all six aforementioned behavioural indicators

## Setup

### Backend

These instructions are for Ubuntu Linux. The steps can be adapted for all major platforms.

> **this node server requires:**
> - [Express](https://expressjs.com/en/starter/installing.html)
> - [body-parser](https://www.npmjs.com/package/body-parser)
> - [mongoose](https://mongoosejs.com/docs/index.html)
> - [mongodb](https://www.npmjs.com/package/mongodb)
> - [morgan](https://www.npmjs.com/package/morgan)

**node.js server**

First you will need an node.js server this can be obtained and installed following the instructions on this [website](https://nodejs.org/en/download/).

After the node.js server is installed you'll need to install (and probably save) the required library as listed in the requirements above.

```
// Check if node is installed
which node
```

**mongodb**

You need also to install [MongoDB](https://www.mongodb.com) executing the four steps of the [MongoDB installation instructions](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition) 

```
// Check if MongoDB is running
mongo
// You should see the mongo client connect to the MongoDB server and show its version number.
// Exit the client using:
> exit
```

**Configuration of the server**

All the server code, including it's configurations is in the server.js file. In this file there are a few settings you may need to change in order to run the server. This server runs only on https and this port can be configured on line 338. *Default server port:*

```
// port:
app.listen("8080", function(){
```

If you need to change the request settings this can be done at line 49 and 50 in server.js.

*Default server request settings:*

```
// header settings
res.header("Access-Control-Allow-Origin", "*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, X-Auth-Token, Content-Type, Accept, Authorization, X-CSRFToken, chap, seq, vert");
```

In order to use the https option a valid http certificate and key are needed. In our case, we purchased a domain alongside an SSL key and certificate and we binded the domain address with the server address of the remote server. 


To build the connection string to the mongoDB you can change this url on line 25 with your own username, password, server_address of the remote server that hosts mongodb and mongodb database_name. In case you want to run the server locally on your machine you need to comment line 25 and uncomment line 24 changing this url on line 24 to you database name:

Default server database settings:

```
// Build the connection string
var dbURL = 'mongodb://localhost:27017/RTlog';
// mongodb://<Host>:<Port>/<custom-Database-name>
```

**running the server**

after everything is configured use the ```node server.js``` command to run the server. If this doesn't work check on the node.js documentation if the running method changed. If everything goes well the following message should appear: ```Server is up!```

If you need to mange multiple servers I recomend that you take a look at [pm2](http://pm2.keymetrics.io)

Now that the server is running it is time to run the [Offline_Computation](https://github.com/gatou92/RealTimeLearningTracker/tree/master/Offline_Computation) code in order to compute the succesful learner profiles and insert them into MongoDB.

### Offline_Computation

For this part you need to download and install [Python (Version 2.7)](https://www.python.org/downloads/)

**Steps for Building the mysql Database**

After having access to the daily log traces of the learners in a previous run of the course, follow the instructions [here](https://github.com/AngusGLChen/DelftX-Daily-Database) in order to build the initial mysql database. 

**config**

- In the [sql2mongodb](https://github.com/gatou92/RealTimeLearningTracker/blob/master/Offline_Computation/sql2mongodb.py) file some lines need to be set.

In line 19:

```
\\put the path to your private rsa key
mypkey = paramiko.RSAKey.from_private_key_file(home + '/.ssh/id_rsa')
```

In lines 22 and 23:

```
\\ put your own username, password, server_address of the remote server that hosts mongodb database and mongodb database_name
client = MongoClient("mongodb://username:password@server_address:27017/database_name")

\\ put your mongodb database_name
db  = client.database_name 
```


### edX_MOOC_pages

**config**

The most important setting is the server address. This needs to be set in the [edX_integration](https://github.com/gatou92/RealTimeLearningTracker/blob/master/edX_MOOC_pages/public/edX_integration.html) file on the line 51. 

```
//replace it with your domain address
var SERVER_URL = "https://mariagatou2.com"; 
```

**static imports**

1. Go in edX Studio and click **Content > Files & Uploads**
2. Click the green " + Upload New File" button
3. Select and upload the **style.css** file from [css](https://github.com/gatou92/RealTimeLearningTracker/tree/master/edX_MOOC_pages/public/css) folder
4. Select and upload the **complex_version.js**, **intermediate_version.js** and **simple_version.js** files from [js](https://github.com/gatou92/RealTimeLearningTracker/tree/master/edX_MOOC_pages/public/js) folder
5. Embody the code of **edX_integration.html** file from folder [public](https://github.com/gatou92/RealTimeLearningTracker/tree/master/edX_MOOC_pages/public) in every MOOC page on edX as raw HTML.

That's it!!! The Real-Time Learning Tracker is set and ready!


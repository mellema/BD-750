## Bodhi Job Engine

#### Introduction
Bodhi Job Engine is a job scheduler running on node.js and MongoDB. The
engine is designed to run scheduled cloud-to-cloud data transactions,
e.g. requesting data from web APIs, pushing data to the Bodhi cloud,
etc.

#### Getting started

Bodhi Job Engine is currently for internal Hot Schedules developers. Jobs
are built on Node.js and are installed as rpms from Artifactory.

Installation of Bodhi Job Engine requires access to the appropriate
Artifactory account. To install:

npm install bodhi-job-engine

In your root directory, add file index.js containing one line of code:

````
var engine = require('bodhi-job-engine');

````

Test your installation by running this command:

````
node index.js

//Expected Output:

//Bodhi Job Engine listening at http://....
//View the UI in a browser at http://....
//No jobs found

````


#### Creating a job

Jobs must be built in a specific format in order for Bodhi Job Engine to
recognize and run them properly. The job should contain an index.js file
that follows this structure:

````
var jobconfig = require('./job-config.js');

exports.config = jobconfig;

exports.run = function(job, done){
    try{
        //do stuff here
        //to post data to Bodhi cloud
        //call the conn method like this where obj is the object holding your data, and 
        //YourType is the name of the type to post your data to:
        jobconfig.conn(obj, jobconfig.agent_config, 'resources/YourType');
        //you must call done() at the end of your code
        done();
    } catch (err) {
        console.log('Error defining job: ', config.job_name, err);
        done(err);
    }
};

````

The job-config.js file should be structured like this:

````
module.exports = {
        //agent_config property required to post to the Bodhi cloud.
    agent_config: {
        target_url: 'https://your_target_url.com',  //where are you posting data to the cloud
        namespace: 'your_bodhi_namespace', //your namespace
        bearerToken: 'your_agent_bearerToken' //alpha numeric agent bearerToken for authentication
    },
    job_name: 'job-app-your_job_name', //the name of your job, must start with 'job-app-'
    time_interval: '3 seconds', //how often do you want the job to run? Understands hours, minutes, seconds, days...
    conn: require('../bodhi-job-engine/app.js').conn
};

````
####Loading a job

Once your job is ready and available in Artifactory, load the job into
the job engine in the root directory of your project using npm install:

````
npm install job-app-your_job_name

````

After installation, edit the node_modules/bodhi-job-engine/engine-config.js to add
your job and enable it (or disable it if necessary):

````
module.exports = {

    jobs: {
        'job-app-your_app_name': {
            enabled: true
        }
    }
};

````

That's it. Starting the job engine should now run your job on the schedule you specified.

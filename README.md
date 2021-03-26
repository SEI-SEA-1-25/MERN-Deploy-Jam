# MERN Deploy Jam

Instructions for how to deploy decoupled MERN apps using MongoDB Atlas, Heroku, and Netlify. 

We will be using a **CD** or *Continuos Deployment* approach to have our apps updated online as new features are completed. This will complement the **CI** or *Continuos Integration* workflow we have been using for this project.

### Overview

* Our Database will be hosted with [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)

> MongoDB Atlas is a cloud hosting service provided by MongoDB. We will have to sign up for an account and configure our server to use MongoDB Atlas when it is deployed.

* We will be using [Heroku](https://www.heroku.com/) once again to host our express server.

> This time around we a going to be using heroku's continuos deployment from github option -- don't worry its super easy

* Our React front end will be deployed with [Netlify](https://www.netlify.com/) and will be configured to speak to our server on heroku.

> Netlify is a cloud computing that offers easy to use frontend deployment hosting and continuos deployment integrations from github


## Configuring the server for deployment

First we will configure the backend for deployment, we need to checkout to a `deployment` branch, so we can keep developing the app with the `main` branch and merge new features into `deployment` when they are ready for production.

* in your server repo:
  * make sure you are on the `main` branch
  * make sure you have the most recent changes from `upstream`
  * make sure you have no uncommited changes
* afterwards use the command `git checkout -b deployment` to create a deployment branch

Now you can update your `.env` with the following environmental variable:
```sh
# in .env
NODE_ENV=development
```

Next we will `NODE_ENV` to configure our server to use different databases depending on the environment (production vs development). Update your mongoose connection in `./models/index.js` to only connect to localhost if `NODE_ENV` is set to `development` with an `if/else`.
It will look something like this:

```js
const mongoose = require('mongoose')

// database connection config
if(process.env.NODE_ENV === 'development') {
  // mongoose config
  const MONGO_URI = process.env.MONGO_URI || 'mongodb://localhost/mernAuth'

  mongoose.connect(MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useCreateIndex: true,
    useFindAndModify: false
  })

  const db = mongoose.connection;

  // db methods for debug
  db.once('open', () => {
    console.log(`⛓ mongoDB connection @ ${db.host}:${db.port}`)
  })

  db.on('error', err => {
    console.error(`🔥 something has gone wrong with the DB!!!!\n ${err}`)
  })

} else {
  // mongoDB Atlas code will go here

  // mongoose config
  mongoose.connect(MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useCreateIndex: true,
    useFindAndModify: false
  })

  const db = mongoose.connection;

  // db methods for debug
  db.once('open', () => {
    console.log(`⛓ mongoDB connection @ ${db.host}:${db.port}`)
  })

  db.on('error', err => {
    console.error(`🔥 something has gone wrong with the DB!!!!\n ${err}`)
  })
}
```

## MongoDB Atlas

We will be using Atlas as the host for our MongoDB database.

*Part 1 - Setting Up Atlas Account:*

### Make an account

Signup for account [here.](https://www.mongodb.com/cloud/atlas) Please note - you will have to use Google Account for OAuth. 

1. Select the "Shared Clusters" free tier. 
2. For cloud provider, select AWS and for region, whichever region is physically closest to you (example: People on the West Coast should choose `us-west-#` ). Leave all other settings on this page at their default. 
3. You will be redirected to a Dashboard where your Clusters will be listed. C

### Create and configure a cluster

(cluster means a mongoDB that lives in the far, far away in the clouds ☁️)

this is our todo list:

- [ ]  Build your first cluster
- [ ]  Create your first database user
- [ ]  Whitelist your IP address
- [ ]  Load Sample Data
- [ ]  Connect your cluster

4. lick the green **Create a New Cluster** button on the right hand side of the page. *PLEASE NOTE: It takes 3-5 min to make a new cluster.* 
5. On the left hand side of the screen there is a hamburger menu. Click on **Database Access** under **Security**  on the lefthand menu and then click the big green button that says  "Add New Database User".
  * For the Authentication Method, leave it on Password. Declare a username and password under Password Authentication. This information will be hidden later in an env variable on your server. **IT IS SUPER IMPORTANT THAT YOU WRITE THIS INFORMATION DOWN**.  
    * *PLEASE NOTE - the fewer special characters your password has, the easier it will be to format your URI call, but also the less secure your db will be* 🤷  **JUST USE REGULAR CHARACTERS FOR THIS DEPLOYMENT**. 
  * Leave all other defaults the same and add user. 
6. click on **Network Access** under **Security**  on the lefthand menu and then click the big green button that says  "Add IP Address".
  * For the sake of this app we are going to allow for access from anywhere since you will each be doing your own Heroku deployments. Click "Allow Access from Anywhere" under "Whitelist a connection IP address." Leave the default `0.0.0.0/0` call, and click "Add IP Address."
7. Return to the Clusters dashboard by clicking **Clusters** under **Data Storage** on the lefthand menu. 
8. On your Cluster, click the Connect button. 
  * Click "Choose a connection method", and on the next page choose "Connect your application."
  * On this new Connect page, under list item two, click "Include full driver code example", and then copy the code MongoDB has created for you into a text file for use in just a moment. 

*Part 2 - Updating Express App to Connect:*

We get to connect our express app to the Cloud!

Head back to your server folder.

1. we need to install the mongodb npm package: `npm i mongodb`
2. Take copied MongoDB code and add to your `index.js` in the else statement we created earlier. 
3. We need to modify the second line of copied code to inlcude our password and then put it in our `.env` file
  * replace \<password> with that password you should have copied down earlier
    *  *Please note - if the password you set for your user has any special characters, you must pass that password in URL encoded format. To format your password for URL encoding use this tool [here.](https://www.urlencoder.org/)* **why'd you do it? I told you not to do it!**
  * add `ATLAS_URI` to your .env file and copy the formatted URI there **WITHOUT QOUTES**
  * update the `const uri` in your index.js to be `const uri = process.env.ATLAS_URI`


```sh
# in .env (fill in the values with your info)
ATLAS_URI=mongodb+srv://super_cool_person:that_password_you_made@cluster0.9hqnh.mongodb.net/myFirstDatabase?retryWrites=true&w=majority
```

```js
// in ./models/index.js
const uri = process.env.ATLAS_URI

```

the formatting of the atlas uri is as follows:

* mongodb+srv://`<user_name>`:`<password>`@cluster0.v70nx.mongodb.net/`<db_name>`?retryWrites=true&w=majority

4. You can test connecting to mongoDB Atlas by updating your `NODE_ENV` to be anything except `development`. Such as `NODE_ENV=banana` or `NODE_ENV=mango` or even just comment it out `# NODE_ENV=development`. After you update your `.env`, kill `nodemon` and restart it.


5. Save, commit your code, and then spin up Nodemon locally and if you are getting your relevant `MONGOOSE CONNECTED` message you are good to move on. **If not, please reach out to a Dev for debugging.** ⭐

6. If your server doesn't crash and you can CRUD your Atlas DB, YOU ARE GOOD TO GO 🎉!

Completed `./models/index.js`

```js
// require mongoose package
const mongoose = require('mongoose')

// database connection config
if(process.env.NODE_ENV === 'development') {
  // mongoose config
  const MONGO_URI = process.env.MONGO_URI || 'mongodb://localhost/mernAuth'

  mongoose.connect(MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useCreateIndex: true,
    useFindAndModify: false
  })

  const db = mongoose.connection;

  // db methods for debug
  db.once('open', () => {
    console.log(`⛓ mongoDB connection @ ${db.host}:${db.port}`)
  })

  db.on('error', err => {
    console.error(`🔥 something has gone wrong with the DB!!!!\n ${err}`)
  })

} else {

  // mongoDB Atlas code will go here
  const MongoClient = require('mongodb').MongoClient;
  
  const uri = process.env.ATLAS_URI

  const client = new MongoClient(uri, { useNewUrlParser: true, useUnifiedTopology: true });
  
  client.connect(err => {
    const collection = client.db("test").collection("devices");
    // perform actions on the collection object
    client.close();
  });

  // connect mongoose
  mongoose.connect(uri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    useCreateIndex: true,
    useFindAndModify: false
  })

  const db = mongoose.connection;

  // db methods for debug
  db.once('open', () => {
    console.log(`⛓ mongoDB connection @ ${db.host}:${db.port}`)
  })

  db.on('error', err => {
    console.error(`🔥 something has gone wrong with the DB!!!!\n ${err}`)
  })
}

```

## Heroku

We will be using Heroku to host our Express Servers that will serve as the API for our frontend to connect to. 

1. Create a Procfile with the command `touch Procfile`
  * Note the capitol `P` in `Procfile`. it is important.
2. In this Procfile add the following code, except replace `server.js` with whichever file contains your Express server declaration (your entry point). After you save `Procfile`, commit your code. 

```sh
web: node server.js
```
3. push your deploy branch to github: git add, git commit and `git push --set-upstream origin deployment`
3. Go to Heroku and login. Create a new app using the top right hand button. Name this app something unique and relevant to your project, but doesn't have to be super specific. 
4. Click Settings on the app, and click Reveal Config Vars to show the equivalent for setting up `.env` variables. 
5. Here, list all the key value pairs that are in your local `.env`
*Note - strings should be passed without quotes. Heroku will format them appropriately.* 
6. Click on the deply tab
  * Under dployment method Select 'connect to github'
  * Serch for your repository name and connect it to heroku
  * after it connects, select the `deployment` branhc to deploy
  * click on 'Enable automatic deploy'

You can can use the heroku CLI with this app, and it will automatically deploy from your repo's deployment branch.

to configure the heroku CLI:
```jsx
heroku login
// this will ask you to click any key
// then will take you to web platform for authentication

heroku git:remote -a name-of-your-heroku-app

heroku logs --tail
```

7. Navigate back to heroku and click on the 'deploy' button
8. Give Heroku 1-2 mins to do the initial build of your app, and then click "open app" in the top right hand corner. If you have any bugs, or if you do not have a home route (`/`) set for your Express app, nothing will render. To debug, type in `heroku logs --tail` into your terminal window and, if needed, grab a Dev to help debug. 🦑

## Netlify

We will be using Netlify to deploy our React frontend, and this frontend will be our Client URL for the backend API calls.

1. At this point, double check that your own personal repo is up to date with the main git repo for your team, and that all your own personal most recent work has been committed and merged into the main git repo. 
  * once your client repo is up to date, checkout to a deployment branch
2. **If you currently have any non-breaking React errors on the terminal when you deploy the frontend:** We have to let Netlify know we are okay with these. Netlify, by default, is not. Longterm, you will want to correct all these errors and then undo the next bit of instructions, but for deployment today do the following. In your `package.json` on the frontend, replace the current `"build": "react-scripts-build",` with `"build": "CI= react-scripts build",`. Commit and push your changes. 
3. Use your Github to sign up for Netlify [here](https://www.netlify.com/).
4. After login, on your dashboard, click "New site from Git" in the top right hand corner. 
5. Under "Continuous Deployment" click "Github", and authorize Netlify to access your Github repositories. 
6. Under "Deploy settings" leave as default, and 
7. To add environmental variables (like your `REACT_APP_SERVER_URL`) click on the "Advanced options" above the Deploy button, then click the "Add variable" button. This should look a lot like the Heroku site. Add your variables and press deploy!
8. Netlify will take several minutes to build the app; check in with a Dev if there are any bugs. 

A few things to note:

- Netlify's environmental variables are set on deploy—if your variable changes, you'll need to trigger a redeploy to make that change go into effect.Instructions for how to deploy decoupled MERN apps using MongoDB Atlas, Heroku, and Netlify. 

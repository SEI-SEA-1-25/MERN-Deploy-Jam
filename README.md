Instructions for how to deploy decoupled MERN apps using MongoDB Atlas, Heroku, and Netlify. 

### MongoDB Atlas

We will be using Atlas as the host for our MongoDB database. 

*Part 1 - Setting Up Atlas Account:*

1. Signup for account [here.](https://www.mongodb.com/cloud/atlas) Please note - you will have to use Google Account for OAuth. 
2. Select the "Shared Clusters" free tier. 
3. For cloud provider, select AWS and for region, whichever region is physically closest to you (example: People on the West Coast should choose `us-west-#` ). Leave all other settings on this page at their default. 
4. You will be redirected to a Dashboard where your Clusters will be listed. On the bottom left hand a modal popup should appear with a list of x5 checklist items:
    - [ ]  Build your first cluster
    - [ ]  Create your first database user
    - [ ]  Whitelist your IP address
    - [ ]  Load Sample Data
    - [ ]  Connect your cluster

    If the modal does not appear, click "Get started" in that same bottom lefthand corner. We are going to be working through some of these list items, but not all.

5. Click on the todo list item "Create your first database user". If it does not load a link, click on **Database Access** under **Security**  on the lefthand menu. 
6. In Database access page, click Add New Database User. For the Authentication Method, leave it on Password. Declare a username and password under Password Authentication. This information will be hidden later in an env variable, and please write down what you have put for future use.  *PLEASE NOTE - the fewer special characters your password has, the easier it will be to format your URI call, but also the less secure your db will be* 🤷 . Leave all other defaults the same and add user. 
7. Return to the Clusters dashboard by clicking Clusters under Data Storage on the lefthand menu. 
8. On your Cluster, click the Connect button. 
9. For the sake of this app we are going to allow for access from anywhere since you will each be doing your own Heroku deployments. Click "Allow Access from Anywhere" under "Whitelist a connection IP address." Leave the default `0.0.0.0/0` call, and click "Add IP Address."
10. Click "Choose a connection method", and on the next page choose "Connect your application."
11. On this new Connect page, under list item two, click "Include full driver code example", and then copy the code MongoDB has created for you. 

*Part 2 - Updating Express App to Connect:*

1. Take copied MongoDB code and add to your `server.js` or `index.js` - or whichever parent file is being used for your Express server initial decleration. Paste right below your db import line (for example `const db = process.env.MONGOD_URI`). 
2. Copy the second line (`const uri = "mongodb+srv://admin:<password>@cluster0.v70nx.mongodb.net/<dbname>?retryWrites=true&w=majority";`), and then delete from your code. Paste as `MONGOD_URI=` in your `.env` file. Example: `MONGOD_URI="mongodb+srv://admin:<password>@cluster0.v70nx.mongodb.net/<dbname>?retryWrites=true&w=majority"`. 
3. Run `npm i mongodb` in your terminal to install mongodb to your project. 
4. In your `.env` file update the key term `<password>` in `MONGOD_URI` to equal whichever password you set for your user in Part 1 of our MongoDB Atlas setup. **Please note - if the password you set for your user has any special characters, you must pass that password in URL encoded format. To format your password for URL encoding use this tool [here.](https://www.urlencoder.org/)**
5. Replace `<dbname>` with whatever you named your Cluster. Default name, if you didn't update, is `Cluster0`. 
6. Update your `db` declaration on your `server.js` or equivalent to point to this `.env` variable, and then make sure that you are passing that variable to the new instance of MongoClient. Example of how this should look here: 

    ```jsx
    const uri = process.env.MONGOD_URI

    // connect to db
    const MongoClient = require('mongodb').MongoClient;
    const client = new MongoClient(uri, { useNewUrlParser: true });
    client.connect(err => {
      const collection = client.db("test").collection("devices");
      // perform actions on the collection object
      client.close();
    });

    mongoose.connect(uri).then((() => console.log('MONGOOSE CONNECTED'))).catch(error => console.log(error))
    ```

7. Save, commit your code, and then spin up Nodemon locally and if you are getting your relevant `MONGOOSE CONNECTED` message you are good to move on. **If not, please reach out to a Dev for debugging.** ⭐

### Heroku

We will be using Heroku to host our Express Servers that will serve as the API for our frontend to connect to. 

1. Go to Heroku and login. Create a new app using the top right hand button. Name this app something unique and relevant to your project, but doesn't have to be super specific. 
2. Click Settings on the app, and click Reveal Config Vars to show the equivalent for setting up `.env` variables. 
3. Here, list all the key value pairs that are in your local `.env`
*Note - strings should be passed without quotes. Heroku will format them appropriately.* 
4. Return to the local instance of your Express server and create a `Procfile`.
5. In this Procfile add the following code, except replace `server.js` with whichever file contains your Express server declaration (your entry point). After you save `Procfile`, commit your code. 

    `web: node server.js`

6. In the command line, type the following commands:

    ```jsx
    heroku login
    // this will ask you to click any key
    // then will take you to web platform for authentication

    heroku git:remote -a name-of-your-heroku-app

    git push heroku main
    ```

7. Give Heroku 1-2 mins to do the initial build of your app, and then click "open app" in the top right hand corner. If you have any bugs, or if you do not have a home route (`/`) set for your Express app, nothing will render. To debug, type in `heroku logs --tail` into your terminal window and, if needed, grab a Dev to help debug. 🦑

### Netlify

We will be using Netlify to deploy our React frontend, and this frontend will be our Client URL for the backend API calls. . 

1. Use your Github to sign up for Netlify [here](https://www.netlify.com/).
2. After login, on your dashboard, click "New site from Git" in the top right hand corner. 
3. At this point, double check that your own personal repo is up to date with the master git repo for your team, and that all your own personal most recent work has been committed and merged into the main git repo. 
4. Under "Continuous Deployment" click "Github", and authorize Netlify to access your Github repositories. 
5. Under "Deploy settings" leave as default, and **pause here.** 
6. **If you currently have any non-breaking React errors on the terminal when you deploy the frontend:** We have to let Netlify know we are okay with these. Netlify, by default, is not. Longterm, you will want to correct all these errors and then undo the next bit of instructions, but for deployment today do the following. In your `package.json` on the frontend, replace the current `scripts: { "build": "react-scripts-build",}` with `"build": "CI= react-scripts build"`. Commit and push your changes. 
7. To add environmental variables (like your server url) click on the "Advanced options" above the Deploy button, then click the "Add variable" button. This should look a lot like the Heroku site. Add your variables and press deploy!
8. Netlify will take several minutes to build the app; check in with a Dev if there are any bugs. 

A few things to note:

- Netlify's environmental variables are set on deploy—if your variable changes, you'll need to trigger a redeploy to make that change go into effect.Instructions for how to deploy decoupled MERN apps using MongoDB Atlas, Heroku, and Netlify. 

### MongoDB Atlas

We will be using Atlas as the host for our MongoDB database. 

*Part 1 - Setting Up Atlas Account:*

1. Signup for account [here.](https://www.mongodb.com/cloud/atlas) Please note - you will have to use Google Account for OAuth. 
2. Select the "Shared Clusters" free tier. 
3. For cloud provider, select AWS and for region, whichever region is physically closest to you (example: People on the West Coast should choose `us-west-#` ). Leave all other settings on this page at their default. 
4. You will be redirected to a Dashboard where your Clusters will be listed. On the bottom left hand a modal popup should appear with a list of x5 checklist items:
    - [ ]  Build your first cluster
    - [ ]  Create your first database user
    - [ ]  Whitelist your IP address
    - [ ]  Load Sample Data
    - [ ]  Connect your cluster

    If the modal does not appear, click "Get started" in that same bottom lefthand corner. We are going to be working through some of these list items, but not all.

5. Click on the todo list item "Create your first database user". If it does not load a link, click on **Database Access** under **Security**  on the lefthand menu. 
6. In Database access page, click Add New Database User. For the Authentication Method, leave it on Password. Declare a username and password under Password Authentication. This information will be hidden later in an env variable, and please write down what you have put for future use.  *PLEASE NOTE - the fewer special characters your password has, the easier it will be to format your URI call, but also the less secure your db will be* 🤷 . Leave all other defaults the same and add user. 
7. Return to the Clusters dashboard by clicking Clusters under Data Storage on the lefthand menu. 
8. On your Cluster, click the Connect button. 
9. For the sake of this app we are going to allow for access from anywhere since you will each be doing your own Heroku deployments. Click "Allow Access from Anywhere" under "Whitelist a connection IP address." Leave the default `0.0.0.0/0` call, and click "Add IP Address."
10. Click "Choose a connection method", and on the next page choose "Connect your application."
11. On this new Connect page, under list item two, click "Include full driver code example", and then copy the code MongoDB has created for you. 

*Part 2 - Updating Express App to Connect:*

1. Take copied MongoDB code and add to your `server.js` or `index.js` - or whichever parent file is being used for your Express server initial decleration. Paste right below your db import line (for example `const db = process.env.MONGOD_URI`). 
2. Copy the second line (`const uri = "mongodb+srv://admin:<password>@cluster0.v70nx.mongodb.net/<dbname>?retryWrites=true&w=majority";`), and then delete from your code. Paste as `MONGOD_URI=` in your `.env` file. Example: `MONGOD_URI="mongodb+srv://admin:<password>@cluster0.v70nx.mongodb.net/<dbname>?retryWrites=true&w=majority"`. 
3. Run `npm i mongodb` in your terminal to install mongodb to your project. 
4. In your `.env` file update the key term `<password>` in `MONGOD_URI` to equal whichever password you set for your user in Part 1 of our MongoDB Atlas setup. **Please note - if the password you set for your user has any special characters, you must pass that password in URL encoded format. To format your password for URL encoding use this tool [here.](https://www.urlencoder.org/)**
5. Replace `<dbname>` with whatever you named your Cluster. Default name, if you didn't update, is `Cluster0`. 
6. Update your `db` declaration on your `server.js` or equivalent to point to this `.env` variable, and then make sure that you are passing that variable to the new instance of MongoClient. Example of how this should look here: 

    ```jsx
    const uri = process.env.MONGOD_URI

    // connect to db
    const MongoClient = require('mongodb').MongoClient;
    const client = new MongoClient(uri, { useNewUrlParser: true });
    client.connect(err => {
      const collection = client.db("test").collection("devices");
      // perform actions on the collection object
      client.close();
    });

    mongoose.connect(uri).then((() => console.log('MONGOOSE CONNECTED'))).catch(error => console.log(error))
    ```

7. Save, commit your code, and then spin up Nodemon locally and if you are getting your relevant `MONGOOSE CONNECTED` message you are good to move on. **If not, please reach out to a Dev for debugging.** ⭐

### Heroku

We will be using Heroku to host our Express Servers that will serve as the API for our frontend to connect to. 

1. Go to Heroku and login. Create a new app using the top right hand button. Name this app something unique and relevant to your project, but doesn't have to be super specific. 
2. Click Settings on the app, and click Reveal Config Vars to show the equivalent for setting up `.env` variables. 
3. Here, list all the key value pairs that are in your local `.env`
*Note - strings should be passed without quotes. Heroku will format them appropriately.* 
4. Return to the local instance of your Express server and create a `Procfile`.
5. In this Procfile add the following code, except replace `server.js` with whichever file contains your Express server declaration (your entry point). After you save `Procfile`, commit your code. 

    `web: node server.js`

6. In the command line, type the following commands:

    ```jsx
    heroku login
    // this will ask you to click any key
    // then will take you to web platform for authentication

    heroku git:remote -a name-of-your-heroku-app

    git push heroku main
    ```

7. Give Heroku 1-2 mins to do the initial build of your app, and then click "open app" in the top right hand corner. If you have any bugs, or if you do not have a home route (`/`) set for your Express app, nothing will render. To debug, type in `heroku logs --tail` into your terminal window and, if needed, grab a Dev to help debug. 🦑

### Netlify

We will be using Netlify to deploy our React frontend, and this frontend will be our Client URL for the backend API calls. . 

1. Use your Github to sign up for Netlify [here](https://www.netlify.com/).
2. After login, on your dashboard, click "New site from Git" in the top right hand corner. 
3. At this point, double check that your own personal repo is up to date with the master git repo for your team, and that all your own personal most recent work has been committed and merged into the main git repo. 
4. Under "Continuous Deployment" click "Github", and authorize Netlify to access your Github repositories. 
5. Under "Deploy settings" leave as default, and **pause here.** 
6. **If you currently have any non-breaking React errors on the terminal when you deploy the frontend:** We have to let Netlify know we are okay with these. Netlify, by default, is not. Longterm, you will want to correct all these errors and then undo the next bit of instructions, but for deployment today do the following. In your `package.json` on the frontend, replace the current `scripts: { "build": "react-scripts-build",}` with `"build": "CI= react-scripts build"`. Commit and push your changes. 
7. To add environmental variables (like your server url) click on the "Advanced options" above the Deploy button, then click the "Add variable" button. This should look a lot like the Heroku site. Add your variables and press deploy!
8. Netlify will take several minutes to build the app; check in with a Dev if there are any bugs. 

A few things to note:

- Netlify's environmental variables are set on deploy—if your variable changes, you'll need to trigger a redeploy to make that change go into effect.

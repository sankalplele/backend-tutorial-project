# Chai aur backend series

Here I will be creating my first backend project under the guidance of Histesh Choudhary Sir, following his "Chai aur Backend" playlist on his YouTube channel "Chai aur Code".

First Lecture

1. Basic Setup of the project like directories for example src, inside which we have middlewares, controllers, db etc. and .env files, .gitignore files to specify which need to be ignored by git.

2. Nodemon Dev Dependency is installed using "npm i -D nodemon" where 'D' represents devDependency

3. Prettier was installed and configured as a dev dependency which helps us write consistent code, if we are using different text editors or multiple people are working on the same project. We added .prettierrc and .prettierIgnore files to configure prettier and keep a track of which files need to be ignored by prettier for formatting, respectively.

Second Lecture

1. We tried connecting to MongoDB Atlas Database. We created an index.js in db folder and used mongoose to connect to the database using the env variable of DB URL. We ensured that this connection is governed by async-await and error is handled using try catch, so that we can debug easily whenvever there is a DB connection error.

2. In our main index.js file, we simply imported the connect db function and in the import statement we ensured that proper path was specified after the 'from' keyword i.e. "./db/index.js" and not ./db/index because it was leading to file not found errors. We simply called the connect db function here.

3. Before calling the connect DB function we need to ensure that the env variables are setup beforehand so we configure dotenv before the connect db to ensure all the env variables are received wherever they are being used.

4. Import  dotenv needs 'require' syntax but for code consistency we need to have it imported through import statements. To solve this, in the 'script' key of package.json, we updated the dev command and added an experimental option to it to solve this problem.

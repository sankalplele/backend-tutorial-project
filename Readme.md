# Chai aur backend series

Here I will be creating my first backend project under the guidance of Histesh Choudhary Sir, following his "Chai aur Backend" playlist on his YouTube channel "Chai aur Code".

> Please explore the documentations of technologies like Express, mongoose etc.

## First Lecture

### Basic Setup

1. Basic Setup of the project like directories for example src, inside which we have middlewares, controllers, db etc. and .env files, .gitignore files to specify which need to be ignored by git.

2. Nodemon as a **Dev Dependency** is installed using `npm i -D nodemon` where 'D' represents devDependency

3. Prettier was installed and configured as a dev dependency which helps us write consistent code, if we are using different text editors or multiple people are working on the same project. We added .prettierrc and .prettierIgnore files to configure prettier and keep a track of which files need to be ignored by prettier for formatting, respectively.

## Second Lecture

### DB Connection index.js and .env intial setup

1. We tried connecting to MongoDB Atlas Database. We created an index.js in db folder and used mongoose to connect to the database using the env variable of DB URL. We ensured that this connection is governed by async-await and error is handled using try catch, so that we can debug easily whenvever there is a DB connection error.

2. In our main index.js file, we simply imported the connect db function and in the import statement we ensured that proper path was specified after the 'from' keyword i.e. "./db/index.js" and not ./db/index because it was leading to file not found errors. We simply called the connect db function here.

3. Before calling the connect DB function we need to ensure that the env variables are setup beforehand in the `index.js` file so we configure dotenv before the connect db to ensure all the env variables are received wherever they are being used.

4. Import dotenv needs `require` syntax but for code consistency we need to have it imported through import statements. To solve this, in the `script` key of `package.json`, we updated the dev command and added an experimental option to it to solve this problem.

In our `package.json` we add this command for using the experimental feature of using `import` syntax for `dotenv`.

```
"scripts": {
  "dev": "nodemon -r dotenv/config --experimental-json-modules src/index.js"
},
```

This is how we configure `dotenv`:

```
import dotenv from "dotenv";
import connectDB from "./db/index.js";

dotenv.config({
  path: "./env",
});
```

## Third Lecture

### app.js setup to use Express JS

1. Coding the app.js file using Express JS. We Imported express, cookie-parser, and cors from npm, in app.js. We then used app.use() to configure things like cors and express.json, especially middlewares.

### DB Connect using Express App

2. Before doing all this we updated our index.js file where connectDB was called and since we know an async function returns a promise, we used == .then() and .catch() on connectDB() == to code app.listen(), i.e, to start our server when the DB is connected.

Our `index.js` file:

```
import dotenv from "dotenv";
import connectDB from "./db/index.js";
import { app } from "./app.js";

dotenv.config({
  path: "./env",
});

connectDB()
  .then(() => {
    app.on("error", (error) => {
      console.log("ERR: ", error);
      throw error;
    });
    app.listen(process.env.PORT || 8000, () => {
      console.log(`Server is running at port : ${process.env.PORT}`);
    });
  })
  .catch((err) => console.log("MONGO DB Connection FAILED", err));
```

3. We configured express.urlencoded to handle urlencoding

4. ### Discussion on **Middlewares** :

> [!NOTE]
> app.use([path,] callback [, callback...])
> Mounts the specified middleware function or functions at the specified path: the middleware function is executed when the base of the requested path matches path.

> Error-handling middleware

Error-handling middleware always takes four arguments. You must provide four arguments to identify it as an error-handling middleware function. Even if you don’t need to use the next object, you must specify it to maintain the signature. Otherwise, the next object will be interpreted as regular middleware and will fail to handle errors. For details about error-handling middleware, see: Error handling.

Define error-handling middleware functions in the same way as other middleware functions, except with four arguments instead of three, specifically with the signature **_`(err, req, res, next)`_**:

```
app.use((err, req, res, next) => {
console.error(err.stack)
res.status(500).send('Something broke!')
})
```

We can use the next method to specify the order of the middlewares execution, i.e. one after the another:

```
app.use((req, res, next) => {
  console.log('Time: %d', Date.now())
  next()
})
```

### Async Handler Utility

5. We will be doing plethora of async calls along with try-catch syntax, so it is super-intuitive to create a wrapper function as a utility to reduce redundancy.

```
const asyncHandler = (requestHandler) => {
  (req, res, next) => {
    Promise.resolve(requestHandler(req, res, next)).catch((err) => next(err));
  };
};
```

> [!IMPORTANT]
> This extra step of asyncHandler is done only to handle errors, because async functions don’t automatically catch errors in Express
>
> ```
> app.get("/user/:id", async (req, res) => {
>  const user = await someDBCall(); // ← Suppose this throws an error
>  res.json(user);
> });
> ```
>
> If someDBCall() throws an error or the await fails:
> The promise is rejected.
> But Express doesn’t know how to handle that rejection.
> The server will crash or hang silently unless you catch the error manually with try-catch.
> That’s where asyncHandler comes in, if we werent using asynchandler function we would then need to use try catch every time we make, for example, a get request like follows:

```
app.get("/user/:id", async (req, res, next) => {
  try {
    const user = await someDBCall();
    res.json(user);
  } catch (error) {
    next(error); // or handle it here
  }
});
```

### ApiError Utility

6. In our utilities folder we create an `ApiError.js` file where we created an `ApiError` class which extends `Error` class and added custom functionality to handle error.
   This is done to standardize error-handling.

> [!NOTE]
> Read `Error` class in Node.js documentation.
> Also learn about JS OOPS concepts like super()
> Also interesting note that `class` keyword was introduced in ES6 (2015)

```
class ApiError extends Error {
  constructor(
    statusCode,
    message = "Something went wrong",
    error = [],
    stack = ""
  ) {
    super(message);
    this.statusCode = statusCode;
    this.data = null;
    this.message = message;
    this.success = false;
    this.errors = errors;

    if (stack) {
      this.stack = stack;
    } else {
      Error.captureStackTrace(this, this.constructor);
    }
  }
}

export { ApiError };
```

> I didnt understand very well this code

### ApiResponse Utility

6. Then we created ApiResponse class in the utilities, and there the highlight was the status codes of an HTTP Response
   This ApiResponse class was created to standardize the structure of the API responses:

```
class ApiResponse {
  constructor(statusCode, data, message = "Success") {
    this.statusCode = statusCode;
    this.data = data;
    this.message = message;
    this.success = statusCode < 400;
  }
}
```

> Learn about status codes

7. We then need to apply checks so that all errors go through the ApiError utitlity hence created in the next lecture.

## Fourth Lecture

### Creating Models using mongoose

Now we go on to create models

> Reference
> https://app.eraser.io/workspace/YtPqZ1VogxGy1jzIDkzj

1. `user.models.js`: simply used mongoose to define the model.

2. Similarly we created `video.models.js`. We need to install a package called **Mongoose aggregate paginate** for using _Mongoose aggreagation pipelines_

#### Model Aggregation Pipelines

```
npm install mongoose-aggregate-paginate-v2
```

> Learn about Mongoose Agreggation Pipelines

#### Installing BCrypt and JSON Web Token

Then we installed **BCrypt** for hashing and comparing passwords basically and **JWT**(JSON Web Tokens) another cryptographic library

```
npm install bcrypt jsonwebtoken
```

#### Middlewares in mongoose using .pre() and .post()

3.  Just before or after the data is saved, updated, deleted, etc. drom the DB, we can have middlewares, example, in mongoose we have `pre` and `post`. This will help us to process the data for tasks such as hashing, encryption or decryption.

```
userSchema.pre("save", async function (next) {
  if (!this.isModified("password")) return next(); // only call bcrypt.hash if the password is modified
  this.password = bcrypt.hash(this.password, 10); // why `this` is used
  next();
});
```

> [!IMPORTANT]
> BCrypt is a hashing library and not an encryption library.
> It uses a one-way password/payload hashing algorithm and hence is non-reversible, i.e., you can find the original password using the hashed string.
> Then how does it compare; it again applies the hash function on the password to be compared and compared the hash string with the original hashed password.
> Hashing is a one-way, irreversible process used for verifying data integrity.
> You can compare a hash but can’t get the original value back.
> Used in password storage, file verification, digital signatures, etc.
> On the other hand, encryption is reversible i.e. can be decrypted

#### Defining custom methods for our models

4. Here is how we can define custom methods:

```
userSchema.methods.isPasswordCorrect = async function (password){
// we check the password using comparison
return await bcrypt.compare(password, this.password)
}
```

#### JWT Basics

5. **_JSON Web Tokens_**: It is a bearer token, _those who bear receive the data_.
   JWT stands for JSON Web Token.

It is a compact, URL-safe token format used for securely transmitting information between parties (usually client ↔ server).

A JWT looks like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjEyMyIsIm5hbWUiOiJTYW5rYWxwIiwiaWF0IjoxNzE5MjIzMDAwfQ.0qUE9dlA3Y7o8aZlFQ1PBMQT7rgdKQTxR-7vWftEdB8
```

_It has 3 parts, separated by dots:_

**Header** – contains token type and signing algorithm. It is a **Base64URL-encoded JSON**

**Payload** – the data (like user id, email, role, etc.). It is also a **Base64URL-encoded JSON**

**Signature** – created using your secret (e.g. `ACCESS_TOKEN_SECRET` as mentioned in you `.env` file). It is an **encrypted HMAC of header + payload + secret**

JWT is secure if you:

1. Sign it with a strong secret (ACCESS_TOKEN_SECRET).
2. Use https (so token isn't sniffed in transit).
3. Set expiration time (exp claim).
4. Don't store it in localStorage (vulnerable to XSS); prefer http-only cookies

> [!IMPORTANT]
> JWT is not encrypted, just signed.
> So you should not store sensitive info (like passwords) in the payload.

**`ACCESS_TOKEN_EXPIRY`=1d** is set up to set the expiry of JWT tokens.

**`REFRESH_TOKEN_SECRET`**: It is a secret key used to sign your refresh tokens, similar to how `ACCESS_TOKEN_SECRET` is used for signing access tokens.

> **_Access Token_** > _Purpose_: Short-lived, used for accessing protected APIs
> _Signed with_: ACCESS\*TOKEN\*SECRET
> **_Refresh Token_** > \_Purpose\*: Long-lived, used to get new access tokens after the old one expires
> \_Signed with\*: REFRESH_TOKEN_SECRET

**`REFRESH_TOKEN_EXPIRY`=10d # Comment: means 10 days**

## Fifth Lecture

#### File Upload using Cloudinary and Multer

We will use a service called **Cloudinary** to store the image and video files and would get a link to access them.

We will be using middleware for uploading this file. We could use either **express file-upload** or **Multer**.
In this project we will be using **Multer** as a middleware to upload the files.

##### Strategy

> 1. Using **Multer**, we upload file on our local server, temporarily.
> 2. Thereafter, we upload it on the cloud.

This is usually done in production-grade apps, otherwise we could directly upload the file on cloud.

##### Creating `cloudinary.js`

1. We create `cloudinary.js` as a utility to configure **Cloudinary**

```
cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});
```

In this file, we will write the functionality of what will be done when we are having a file on our local server and we want to save it on cloud, i.e., part two of our strategy.

```
import { v2 as cloudinary } from "cloudinary";
import fs from "fs";

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
});

const uploadOnCloudinary = async (localFilePath) => {
  try {
    if (!localFilePath) return null;
    // upload on cloudinary
    const response = await cloudinary.uploader.upload(localFilePath, {
      resource_type: "auto",
    });

    console.log("File uploaded successfully on cloudinary", response.url);
    return response;
  } catch (error) {
    fs.unlinkSync(localFilePath); // remove the locally saved temporary file as the upload operation got failed
    console.error("Cloudinary upload error:", error);
    return null;
  }
};

export { uploadOnCloudinary };

```

##### Creating `multer.middleware.js`

2. Now we create the `multer.middleware.js`:




## SETUP


This part is mainly for the basic setup of the server. We go over the installation of Express and the basic GraphQL setup. We also add MongoDB and we make mocks on it.

1. Create a folder to put the server and mobile folder in
2. Cd into the server folder and run the command `yarn init` and click enter on each question
3. Run `yarn add express cross-env body-parser`
4. Create a folder called `src` and create a file inside called `index.js`
5. Put the basic setup of express at the top of the file

```js
import express from 'express';
import bodyParser from 'body-parser';

const app = express(); // create an instance of express

const PORT = process.env.PORT || 3000; // create the port

app.use(bodyParser.json()); // add body-parser as the json parser middleware

app.listen(PORT, err => {
  if (err) {
    console.error(err);
  } else {
    console.log(`App listen on port: ${PORT}`);
  }
});
```
5. Now we need to install babel in the devDependencies because we want to write in latest javascript feature.

`yarn add -D babel-cli babel-plugin-transform-object-rest-spread babel-preset-env`

6. Go into your `package.json` and add this script under the scripts:

```json
"dev": "cross-env NODE_ENV=dev nodemon --exec babel-node src/index.js"
```

7. Create a `.babelrc` with these settings:

```
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "node": "6.10"
        }
      }
    ]
  ],
  "plugins": [
    [
      "transform-object-rest-spread",
      {
        "useBuiltIns": true
      }
    ]
  ]
}
```

Here I make use of `babel-preset-env`. This helps us to setup babel without pain.

8. Now it is time to set up some GraphQL stuff. First create a folder called `graphql` inside `src`. After that create a `schema.js` file and put the follow in it:

```js
export default`
  type Tweet {
    _id: String
    text: String
  }

  type Query {
    getTweets: [Tweet]
  }

  schema {
    query: Query
  }
`;
```

Here we have a basic schema for the tweet. Also we have a query call `getTweets` you render a list of this tweet.

9. Now it's time to write the resolver for the `getTweets`. Create a folder `resolvers` inside `src/graphql/`. After that, create a file called `tweet-resolver.js`. Inside this one put this:

```js
import Tweet from '../../models/Tweet';

export default {
  getTweets: () => Tweet.find({})
}
```

And create a `index.js` file inside `src/graphql/resolvers/` folder and put :

```js
import TweetResolvers from './tweet-resolver';

export default {
  Query: {
    getTweets: TweetResolvers.getTweets
  }
}
```

But where did this `Tweet` models come from? Yes, this is time to setup the db.

10. Create a `db.js` inside the `src/config/` folder. Put this lines in after running this command.

`yarn add mongoose`

```js
/* eslint-disable no-console */

import mongoose from 'mongoose';

import constants from './constants';

mongoose.Promise = global.Promise;

mongoose.set('debug', true); // debug mode on

try {
  mongoose.connect(constants.DB_URL, {
    useMongoClient: true,
  });
} catch (err) {
  mongoose.createConnection(constants.DB_URL, {
    useMongoClient: true,
  });
}

mongoose.connection
  .once('open', () => console.log('MongoDB Running'))
  .on('error', e => {
    throw e;
  });
```

Nothing crazy here just the basic setup of a db with mongodb. But where do the constants file came ? Time to create it. So inside the `src/config/` folder create a file called `constants.js'` and put these lines in.

```js
export default {
  PORT: process.env.PORT || 3000,
  DB_URL: 'mongodb://localhost/tweeter-development',
  GRAPHQL_PATH: '/graphql'
}
```

Here, these are the constants where we put the base configuration of the server.

  - PORT -> the port of the app
  - DB_URL -> url for the db
  - GRAPHQL_PATH -> the url for graphql server

11. Time to create the Mongodb schema of the tweet. Inside `src/` folder create `models` folder and inside this one `Tweet.js` file. Inside this one we create a basic schema for now ;)

```js
import mongoose, { Schema } from 'mongoose';

const TweetSchema = new Schema({
  text: String,
});

export default mongoose.model('Tweet', TweetSchema);
```

12. Time to update the simple express server for add graphql on it inside `src/index.js`

`yarn add apollo-server-express graphql-tools`

```js
import express from 'express';
import { graphqlExpress, graphiqlExpress } from 'apollo-server-express';
import { makeExecutableSchema } from 'graphql-tools';
import { createServer } from 'http';
import bodyParser from 'body-parser';

import './config/db';
import constants from './config/constants';
import typeDefs from './graphql/schema';
import resolvers from './graphql/resolvers';

app.use(bodyParser.json());

app.use(
  '/graphiql',
  graphiqlExpress({
    endpointURL: constants.GRAPHQL_PATH,
  }),
);

app.use(
  constants.GRAPHQL_PATH,
  graphqlExpress({
    schema
  }),
);

graphQLServer.listen(constants.PORT, err => {
  if (err) {
    console.error(err);
  } else {
    console.log(`App running on port: ${constants.PORT}`);
  }
});
```

Nothing crazy here, basic setup + add graphiql the IDE of grahpql.

13. Time to mocks some stuff ? Sure ;) That's gonna be easy, first think add `yarn add -D faker` which is a library to help you with mock data. We will create 10 fake mock tweet.

Inside `src` create a folder `mocks` and a file `index.js` inside this one.

```js
import faker from 'faker';

import Tweet from '../models/Tweet';

const TWEETS_TOTAL = 10;

export default async () => {
  try {
    await Tweet.remove();

    await Array.from({ length: TWEETS_TOTAL }).forEach(async () => {
      await Tweet.create({
        text: faker.lorem.paragraphs(1),
      })
    });
  } catch (error) {
    throw error;
  }
}
```

14. Add this mocks promise to the server inside `src/index.js`

```js
// all other import
import mocks from './mocks';

// other code here

mocks().then(() => {
  graphQLServer.listen(constants.PORT, err => {
    if (err) {
      console.error(err);
    } else {
      console.log(`App running on port: ${constants.PORT}`);
    }
  });
});
```

15. Time to test it. Go on https://github.com/skevy/graphiql-app and download the app. Why ? Because we will need this jwt auth later ;)

16. Open this tool now and add this in the left part. This is a simple query where we get all 10 tweets. Don't forget to put the url `http://localhost:3000/graphql`

```js
{
  getTweets {
    _id
		text
  }
}
```


#### OUTPUT 1

<img src="https://image.ibb.co/nGHnpv/Screen_Shot_2017_08_03_at_8_31_10_PM.png" height="75%" width="75%" border="0">


---

#### OUTPUT 2

<img src="https://preview.ibb.co/eKsdWa/Screen_Shot_2017_08_05_at_10_57_43_AM.png" alt="Screen_Shot_2017_08_05_at_10_57_43_AM" border="0">


#### OUTPUT 4

<img src="https://image.ibb.co/iX2rNF/Screen_Shot_2017_08_09_at_8_02_22_AM.png" alt="Screen_Shot_2017_08_05_at_10_57_43_AM" border="0">
<img src="https://image.ibb.co/j27QhF/Screen_Shot_2017_08_09_at_8_02_25_AM.png" alt="Screen_Shot_2017_08_05_at_10_57_43_AM" border="0">


### OUTPUT 5


<img src="https://image.ibb.co/jrQBcF/Screen_Shot_2017_08_15_at_8_43_24_AM.png" alt="Screen_Shot_2017_08_05_at_10_57_43_AM" border="0">

<img src="https://image.ibb.co/dLYKWa/Screen_Shot_2017_08_15_at_8_43_29_AM.png" alt="Screen_Shot_2017_08_05_at_10_57_43_AM" border="0">

---

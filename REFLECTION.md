## Q1 - Middleware

Middleware consists of functions that run when the back-end server gets requests. It can edit requests, or decide whether the request should be forwarded to the route handler. The order matters, for example, in our code we need to have `express.json()` before the routes so that we parse the JSON correctly before trying to use things like `req.body` in the routes. 

Global middleware applies to every request that goes to the server, it is registered with `app.use()`. For example, `app.use(cors(...))` adds the CORS header to all of our responses. 

Route-level middleware only runs for a specific route. For example, `app.post('/api/scores',verifyToken, ...)` only verifies the token for that specific route. 

## Q2 - Password Security

Passwords should never be stored in plaintext because the reality is that companies get hacked and databases get leaked. In case that your company gets hacked, you do not want the attackers to have all of the users' passwords. Instead, we can hash them such that the attackers cannot read them in plain text, but we can still use them to authenticate the users. When we call `bcrypt.hash(password,10)`, we are hashing the password using the `bcrypt` library and then `10` represents the number of rounds of hashing. In this case, we hash the password `2^10` times over. 

## Q3 - JWT Flow

When registering an account, the client sends a `POST /api/auth/register` containing an email and password. The server hashes the password with `bcrypt.hash(password,10)`, and creates a new user in MongoDB. The server returns a `201 Created` with the `userID` and email. 

When logging in, the client sends a `POST /api/auth/login` containing the email and password. The server finds a user with the email, compares the password using `bcrypt.compare(password, user.passwordHash)`, and if valid, creates a valid JWT. 

When submitting a score, the user sends a `POST /api/scores` with the authorization header `Authorization: Bearer <JWT-token>`. The server verifies the token, and if it is valid, creates a score using `Score.create(...)`. The server returns a `201 Created` with the JSON of the saved score document. 

Embedded in the JWT is the `userID` and the user's email. The server does not need to do a database lookup for every request because the token contains the user's identity and shows that the request can be trusted. Hence, the server can authenticate the request without having to check the database to see if the user exists. 

## Q4 - In-Memory vs Database

Problems with In-Memory: 
1. Data is lost when the server is restarted, because it is only stored in memory. This is solved by MongoDB because the data is stored in a database which will persist across server restarts. 
2. If we were to expand our application, storing data in the server memory would not work with multiple server instances, meaning that it would be difficult to have multiple back-end servers running for redundancy. This is solved by using a database because multiple servers can easily connect to the same database. 

When redeploying the MongoDB server, the data remains intact because it is stored in a database which actually writes the data to files. Our app will be able to reconnect with the database and see the same scores/data. This is different than in-memory storage because in-memory storage is not written to files and will be lost when the server restarts. 

## Q5 - Route Design Decision

The `GET /api/scores` is public because there is no need to restrict access. It makes sense to let anyone view the leaderboard as long as they cannot change it. The `POST /api/scores` is restricted to authenticated users because only registered users should be able to submit a score, that way we know that scores are associated with real users. 

If `GET /api/scores` also required authentication, then casual visitors to the site would not be able to view the leaderboard, only those who were logged in. 


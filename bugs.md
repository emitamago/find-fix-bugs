- BUG number one
    * middleware/auth.js 
    
    BUG/SECURITY ISSUE
    ```js
     let payload = jwt.decode(token) 
     ```
     FIXED VERSION
    ```js
    let payload = jwt.verify(token, SECRET_KEY)
    ```
    decode does not require the token to be signed and it will lead serious security issue

- BUG number two
    * models/user  `get(username)` It will not return 404 when no user found.  it will not raise error.
    
    BUG VERSION
    ```js
    if (!user) {new ExpressError('No such user', 404); }
    ```
   
    FIXED VERSION
    ```js
    if (!user) {throw new ExpressError('No such user', 404); }
    ```
    Error is needed to be throw.

- BUG number three
    * models/user `update(username, data)` it will update any colums that passed in. Need to check passed data are vailed and throw error if the field in not allowed. Added if statement 
   
    FIXED VERSION
    ```js
    if(data.admin !== undefined ||
        data.username !== undefined ||
        data.password !== undefined){
            throw new ExpressError("Not allowed Fields", 400)
        }
    ```
    There was a test for this but the test passed because of different error. Bug number four was a reason

    - BUG number four
        * routes/user `patch users/:username` Because of middleware, only admin user could update. Non admin user could not update even it own data. Original version below
        
        BUG VERSION
        ```js
        router.patch('/:username', authUser, requireLogin, requireAdmin,async function(
        ```
        third middleware `requireAdmin` was preventing non admin user to enter this route so they could not update own data.
        
        FIXED VERSION
        ```js
        router.patch('/:username', authUser, requireLogin,async function(
        ```
        So the middleware removed.

- BUG number five
    * route/auth `post /auth/login` Anybody could get token when request is made.
    
    BUG VERSION 
    ```js
    let user = User.authenticate(username, password);
    const token = createTokenForUser(username, user.admin);
    ```
    Because `User.authenticate` return just promise here so any request can proceed next line which create token and return.
    
    FIXED VERSION
    ```js
    let user = await User.authenticate(username, password);
    const token = createTokenForUser(username, user.admin);
    ```
    Added to `await` keyword to actually authenticate the user

- BUG number 6
    * route/user `delete/:username` it will not raise any error if ther is not matching user to delete.

    BUG VERSION Because code below is return promise and code json will be return before error raise 
    ```js
    User.delete(req.params.username);
    ```
    FIXED VERSION
    ```js
    await User..delete(req.params.username);
    ```
    Added `await` keyword to actually get result
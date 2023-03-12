# Secrets
In this project I just implemented differents types of Authentication methods.

## Basic Authentication:
- Encription
- Hashing
- I use [Passport](https://www.passportjs.org/) and [Passport-local](https://www.passportjs.org/packages/passport-local/) for implement that.
- And also use [express-session](https://github.com/expressjs/session#readme) for usging session and cookies.
```

const userSchema = new mongoose.Schema({
    email: String,
    password: String,
    googleId: String,
    facebookId: String,
    secret:String
});

userSchema.plugin(passportLocalMongoose);
userSchema.plugin(findOrCreate);


const User = mongoose.model("User", userSchema);

passport.use(User.createStrategy());

passport.serializeUser(function (user, cb) {
    process.nextTick(function () {
        return cb(null, {
            id: user.id,
            username: user.username,
            picture: user.picture
        });
    });
});

passport.deserializeUser(function (user, cb) {
    process.nextTick(function () {
        return cb(null, user);
    });
});

```


## Google Authentication
- I used [passport-google-oauth20](https://www.passportjs.org/packages/passport-google-oauth20/)
- Also have to create an app in [Google Developers Console](https://console.developers.google.com/) for **client ID** and **client secret**.

```
// Configure Strategy
passport.use(new GoogleStrategy({
    clientID: process.env.CLIENT_ID,
    clientSecret: process.env.CLIENT_SECRET,
    callbackURL: "http://localhost:3000/auth/google/secrets"
},
    function (accessToken, refreshToken, profile, cb) {
        console.log(profile);
        User.findOrCreate({ googleId: profile.id }, function (err, user) {
            return cb(err, user);
        });
    }
));

// Authenticate Requests
app.get('/auth/google',
    passport.authenticate('google', { scope: ['profile'] }));

app.get("/auth/google/secrets",
    passport.authenticate('google', { failureRedirect: "/login" }),
    function (req, res) {
        res.redirect("/submit");
    });
```

## Facebook Authentication
- I use [passport-facebook](https://www.passportjs.org/packages/passport-facebook/)
- Also have to create an app in [Facebook Developers](https://developers.facebook.com/) for **client ID** and **client secret**.

```
// Configure Strategy
passport.use(new FacebookStrategy({
    clientID: process.env.FACEBOOK_ID,
    clientSecret: process.env.FACEBOOK_SECRET,
    callbackURL: "http://localhost:3000/auth/facebook/callback"
},
    function (accessToken, refreshToken, profile, cb) {
        User.findOrCreate({ facebookId: profile.id }, function (err, user) {
            return cb(err, user);
        });
    }
));

// Authenticate Requests
app.get('/auth/facebook',
    passport.authenticate('facebook'));

app.get('/auth/facebook/callback',
    passport.authenticate('facebook', { failureRedirect: '/login' }),
    function (req, res) {
        // Successful authentication, redirect home.
        res.redirect("/submit");
    });

```

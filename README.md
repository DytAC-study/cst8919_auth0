# CST8919-lab01-Auth0

demo-video link: https://youtu.be/n7O9Bs1vduM



## 🔧 **1. Setup & Auth0 Configuration**

### ✅ Step 1: Create an Auth0 Account & Application

1. Go to [Auth0](https://auth0.com/) and sign in.
2. Create a new **application**:
   - Type: **Regular Web Application**
   - Name: `Flask Login App` (or similar)
3. Under **Settings**, note the following:
   - `Domain`
   - `Client ID`
   - `Client Secret`
4. Set Allowed URLs:
   - **Allowed Callback URLs**: `http://localhost:3000/callback`
   - **Allowed Logout URLs**: `http://localhost:3000`

------

## 🐍 **2. Flask App Setup**

### ✅ Step 2: Project Structure

```
CST8919_AUTH0/
├── server.py
├── .env
├── requirements.txt
├── templates/
│   ├── dashboard.html
│   └── home.html
├── README.md
├── .gitignore
```

### ✅ Step 3: Start venv & Install Dependencies

start Python venv under Linux for demonstration:

1. setup venv

   ```
   python3 -m venv venv
   ```

2. start venv

   ```
   source venv/bin/activate
   ```

3. exit venv (after demonstration)

   ```
   deactivate
   ```

Install dependencies in venv:

```
pip install -r requirements.txt
```

> **`requirements.txt`**

```
flask>=2.0.3
python-dotenv>=0.19.2
authlib>=1.0
requests>=2.27.1
```

------

## 🔑 **3. Environment Variables**

Create a `.env` file:

```
AUTH0_CLIENT_ID=hoYxAdDw8TOGVO4JwO8UmkJzpFhMnMSd
AUTH0_CLIENT_SECRET=t6epGyzAHW7aXRSPGie59EmnG-kfjFaDCeFPq-RhepiNxD-P4jYiB7n01S4nC9BC
AUTH0_DOMAIN=dev-5lz55m8hd1yeqmj0.us.auth0.com
APP_SECRET_KEY=
```

Generate a suitable string for `APP_SECRET_KEY` using `openssl rand -hex 32` from your shell.

------

## 🧠 **4. Core Flask Logic**

### server.py

Follow the Auth0 Flask Quickstart and implement login/logout using the `authlib` library.

### ✅ Step 4: Update Flask APP

```
import json
from os import environ as env
from urllib.parse import quote_plus, urlencode

from authlib.integrations.flask_client import OAuth
from dotenv import find_dotenv, load_dotenv
from flask import Flask, redirect, render_template, session, url_for

ENV_FILE = find_dotenv()
if ENV_FILE:
    load_dotenv(ENV_FILE)

app = Flask(__name__)
app.secret_key = env.get("APP_SECRET_KEY")

oauth = OAuth(app)

oauth.register(
    "auth0",
    client_id=env.get("AUTH0_CLIENT_ID"),
    client_secret=env.get("AUTH0_CLIENT_SECRET"),
    client_kwargs={
        "scope": "openid profile email",
    },
    server_metadata_url=f'https://{env.get("AUTH0_DOMAIN")}/.well-known/openid-configuration'
)

@app.route("/login")
def login():
    return oauth.auth0.authorize_redirect(
        redirect_uri=url_for("callback", _external=True)
    )


@app.route("/callback", methods=["GET", "POST"])
def callback():
    token = oauth.auth0.authorize_access_token()
    session["user"] = token
    return redirect("/")

@app.route("/logout")
def logout():
    session.clear()
    return redirect(
        "https://" + env.get("AUTH0_DOMAIN")
        + "/v2/logout?"
        + urlencode(
            {
                "returnTo": url_for("home", _external=True),
                "client_id": env.get("AUTH0_CLIENT_ID"),
            },
            quote_via=quote_plus,
        )
    )

@app.route("/")
def home():
    return render_template("home.html", session=session.get('user'), pretty=json.dumps(session.get('user'), indent=4))

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=env.get("PORT", 3000))
```

------

## 📄 **5. Run your application**

You're ready to run your application! From your project directory, open a shell and use:

```sh
python3 server.py
```

Your application should now be ready to open from your browser at [http://localhost:3000](http://localhost:3000/)


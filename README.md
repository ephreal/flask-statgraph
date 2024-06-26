
# Flask-Statgraph

Flask-Statgraph is an extensions that collects data from requests send to the server to then display statistics like response-time, hits, unique hits, etc.

## Installation
You can install the extensions with pip:

    $ pip install flask-statgraph

## Usage
Below is a small example on how to use the extension.
```py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_statgraph import Statistics

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///database.db"

db = SQLAlchemy(app)

class Request(db.Model):
    __tablename__ = "request"

    index = db.Column(db.Integer, primary_key=True, autoincrement=True)
    response_time = db.Column(db.Float)
    date = db.Column(db.DateTime)
    method = db.Column(db.String)
    size = db.Column(db.Integer)
    status_code = db.Column(db.Integer)
    path = db.Column(db.String)
    user_agent = db.Column(db.String)
    remote_address = db.Column(db.String)
    exception = db.Column(db.String)
    referrer = db.Column(db.String)
    browser = db.Column(db.String)
    platform = db.Column(db.String)
    mimetype = db.Column(db.String)

db.create_all()

statistics = Statistics(app, db, Request)

@app.route("/")
def index():
    return "Hello World!"

if  __name__ == "__main__":
    app.run()
```

The script above sets up a basic SQLAlchemy Database.
When initalizing the statistics extensions, you will have to provide a model (here: ```Request```) to store the data in.

Now, whenever a user requests an endpoint, it will be stored in the defined model.

> If you are in debug mode and an exception occures, teardown of the request will not happen and therefore it won't be stored in the database. If you want to teardown it by force, set ```PRESERVE_CONTEXT_ON_EXCEPTION``` to False in the config.

### Things that are stored
|Key|Type|Description  |
|--|--|--|
|response_time|```float```|The time it took the server to process the request.|
|date|```datetime.datetime```|The date the request was send to the server (in UTC).|
|method|```str```|The HTTP method that was used (e.g. GET, POST, ..).|
|size|```int```|The body size in bytes that was send to the client.|
|status_code|```int```|The status code that was returned by the request.
|user_agent|```str```|The User-Agent that was send with the request.|
|remote_address|```str```|The ip address of the client.|
|exception|```str```|If an error occured, this field will have the error message and the status_code will automatically be 500. <br/>Example: ```ZeroDivisionError('division by zero')```|
|referrer|```str```|Link to the website that referred the user to the endpoint.|
|browser|```str```|The browser that was used to send the request. <br/> Example: ```firefox 72.0```|
|platform|```str```|Operating System the request was send from.|
|mimetype|```str```|Mimetype of the response send to the client (e.g. html/text).|

## View the statistics
Flask-Statgraph will add a blueprint with the prefix ```../statistics``` to your app.

![preview_image](https://i.imgur.com/0P3y7E7.png)

### Config
|Name|Type|Description|Default|
|--|--|--|--|
|STATISTICS_DEFAULT_DATE_SPAN|```datetime.timedelta```|If no date was selected, it will be set to today as end and today - option as start.|```datetime.timedelta(days=7)```

## Protect the route
Flask-Statgraph accepts a fourth argument: A function that is called before any requests to the statistics blueprint.

```py
...

def check_if_user_is_allowed():
    # check if user is allowed to
    # e.g. JWT Token, Flask Login etc.
    # ...
    if not allowed:
        return redirect(url_for("index"))

statistics = Statistics(app, db, Request, check_if_user_is_allowed)
```

There you can check if the user has the required permissions to view the statistics. If not, redirect him to another page.

## Disable statistics
If a user has opt-out, request record can be disabled with custom function `disable_f`.
```
statistics = Statistics(app, db, Request, disable_f= lambda: current_user.has_optout() )
```
or
```
def has_disable_cookie():
    return request.cookies.get('disable_stat') == 'True'

statistics = Statistics(app, db, Request, disable_f=has_disable_cookie )
```

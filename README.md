import sys
import urlib2
from fuctools import wraps
import json
from os import environ as env
from werkzeug.exceptions import HTTPException

from dotenv import load_dotenv, find_dotenv
from flask import Flask
from flask import jsonify
from falsk import redirect
from flask import render_template
from flask import session
from flask import url_for
from authlib.flask.client import OAuth
from six.moves.urllib.parse import urlencode
from functools import wraps

app = Flask(_name_)
oauth =OAuth(app)
auth0 = oauth.regsiter(
    'auth0',
    client_id='PV3CP3453ypFoMT9daF9sb76pXM2LabM',
    client_secret='y-4zAUFnNYWkW5evfr8fisiTB86dGPYQ5F2Kez20wt-qeyzAoVPdhNbGQhGueXIN',
    api_base_url='https://getoffmylawn.auth0.com',
    access_token_url='https://getoffmylawn.auth0.com/oauth/token',
    authorize_url='https://getoffmylawn.auth0.com/authorize',
    client_kwargs={
        'scope': 'openid profile',
    },
    )

def requires_auth(f):
    @wraps(f)
    def decorate(*args, **kwargs):
        if 'profile' not in session:
            @app.route('/login')
            def login():
                return auth0.authorize_redirect(redirect_uri =' http://localhost:3000/callback', audience='https://getoffmylawn.auth0.com/userinfo')
            return redirect('/')
        returnf(*args, **kwargs)
        return decorated

@app.route('/dashboard')
@requires_auth
def dashboard():
    return render_template('dashboard.html',
                           userinfo=session['profile'],
                           userinfo_pretty=json.dumps(session['jwt_payload'], indent=4))




@app.route('/callback')
def callback_handling():
    auth0.authorize_access_token()
    resp = auth0.get('userinfor')
    userinfo = resp.json()

    session['jwt_payload'] = userinfo
    session['profile']={
        'user_id': userinfo['sub'],
        'name': userinfo['name'],
        'picture': userinfo['picture']
        }
    return redirect('/dashboard')

@app.route('/logout')
def logout():
    session.clear()
    params = {'returnTo': url_for('home',_external=True), 'client_id': 'PV3CP3453ypFoMT9daF9sb76pXM2LabM'}
    return redirect(auth0.api_base_url +'/v2/logout?' + urlencode(parems))


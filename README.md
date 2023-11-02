# Vulnerabilities Project

## Task

In this project I was given a codebase with a number of security vulnerabilities that needed addressing. My task was to find the issues using various scanning tools (Bearer, Bandit, OWASP) and fix them.

## Vulnerabilities Found and Rectified

**1. Four potentialities for SQL injections:**

Original code sample:
```bash
"INSERT INTO user (username, password) VALUES ('"+username+"', '"+password+"')", ()
```
Rectified code sample:
```bash
"INSERT INTO user (username, password) VALUES (?, ?)", (username, password)
```

**2. Hardcoded secret key:**

Original code sample:
```bash
SECRET_KEY='super_secret_key'
```
Rectified code sample:
```bash
SECRET_KEY=os.environ.get('SECRET_KEY')
```

**3. Absence of CSRF tokens:**

Rectified code sample:
```bash
# Additions to __init__.py
from flask_wtf.csrf import CSRFProtect
csrf = CSRFProtect()
csrf.init_app(app)
# Additions to all forms in the HTML files
<input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>
```

**4. CSP Headers too open:**

Original code sample:
```bash
@app.after_request
    def add_security_headers(resp):
        resp.headers['Content-Security-Policy']='default-src \'self\''
        return resp
```
Rectified code sample:
```bash
@app.after_request
    def add_security_headers(resp):
        resp.headers['Content-Security-Policy'] = (
            "default-src 'self'; "
            "script-src 'self';"
            "img-src 'self';"
            "font-src 'self';"
            "form-action 'self';"
            "frame-ancestors 'none';"
        )
        return resp
```

**5. Lack of samesite protection:**

Rectified code sample:
```bash
# Additions to __init__.py
app.config.update(
        SESSION_COOKIE_SECURE=True,
        SESSION_COOKIE_HTTPONLY=True,
        SESSION_COOKIE_SAMESITE='Strict',
    )
# ......
@app.after_request
def add_security_headers(resp):
    # ...reponse headers...
    resp.set_cookie('cookieName', 'cookieValue', secure=True, httponly=True, samesite='Strict')
    return resp
# Additions to auth.py
@bp.route('/hello')
def hello():
    response = make_response('Hello, World!')
    response.set_cookie('cookieName', 'cookieValue', secure=True, httponly=True, samesite='Strict')
    return response
```

**6. Lack of nosniff protection:**

Rectified code sample:
```bash
# Additions to __init__.py
@app.after_request
def add_security_headers(resp):
    # ...reponse headers...
    # ...resp.set_cookie...
    resp.headers['X-Content-Type-Options'] = 'nosniff'
    return resp
```
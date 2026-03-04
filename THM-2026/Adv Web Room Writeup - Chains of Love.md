## Checking live hosts
Navigating to target ip, seems we need to add domain to host files
![](attachments/Pasted%20image%2020260304113634.png)
![](attachments/Pasted%20image%2020260304113650.png)
![](attachments/Pasted%20image%2020260304113705.png)

Now we have this!


> Enumeration

Tried Directory fuzzing and found some endpoints
![](attachments/Pasted%20image%2020260304123858.png)


>Gaining Access

Couldnt bruteforce my way into the admin page,
![](attachments/Pasted%20image%2020260304123644.png)

Checked the source code n saw sumn interesting, 
![](attachments/Pasted%20image%2020260304123940.png)

Didn't really mean much so i checked the app.py dir i found earlier

```
from flask import Flask, request, render_template, render_template_string, redirect
import os
import jwt
import datetime
import requests
from markupsafe import escape

app = Flask(__name__, static_folder=".", static_url_path="")

JWT_SECRET = os.environ.get("ADMIN_SECRET", "dev_secret_for_ctf")

app.config["DEBUG"] = False
app.config["ENV"] = "production"
app.config["VERSION"] = "2.3.1"
app.config["DATABASE_URL"] = "postgresql://app_user:********@db.internal:5432/novadev"
app.config["REDIS_HOST"] = "redis.internal"
app.config["ADMIN_SECRET"] = JWT_SECRET

ADMIN_USERNAME = "nova_admin_9f3b2c7d8e1a"
ADMIN_PASSWORD = "X7!rM9#Qp2@Lk$W4E8^BzA"

@app.route("/")
def home():
    return render_template("home.html")


@app.route("/about")
def about():
    return render_template("about.html")


@app.route("/services")
def services():
    return render_template("services.html")


@app.route("/contact", methods=["GET", "POST"])
def contact():

    if request.method == "POST":

        message = request.form.get("message", "").strip()

        # Security by Obscurity

        if message == "{{ config }}":

            return render_template_string(
            message,
            config=app.config
            )

        #This escapes all text
        safe_message = escape(message)

        template = f"""
        <h3>Thank you for your message</h3>
        <div class="preview-box">
            {safe_message}
        </div>
        """

        return template
    return render_template("contact.html")

def verify_jwt(token):
    try:
        return jwt.decode(token, JWT_SECRET, algorithms=["HS256"])
    except Exception:
        return None

@app.route("/admin/login", methods=["GET", "POST"])
def admin_login():
    if request.method == "POST":
        username = request.form.get("username")
        password = request.form.get("password")

        if username == ADMIN_USERNAME and password == ADMIN_PASSWORD:
            payload = {
                "user": username,
                "role": "admin",
                "iat": datetime.datetime.utcnow(),
                "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=1)
            }

            token = jwt.encode(payload, JWT_SECRET, algorithm="HS256")

            resp = redirect("/admin")
            resp.set_cookie("token", token)
            return resp

        return render_template("admin_login.html", error="Invalid credentials")

    return render_template("admin_login.html")

@app.route("/admin")
def admin():
    token = request.cookies.get("token")
    if not token:
        return redirect("/admin/login")

    data = verify_jwt(token)
    if not data or data.get("role") != "admin":
        return "Unauthorized"

    return render_template("admin.html")


@app.route("/admin/fetch")
def fetch():
    token = request.cookies.get("token")
    data = verify_jwt(token)

    if not data or data.get("role") != "admin":
        return "Unauthorized"

    url = request.args.get("url")
    if not url:
        return "No URL provided"

    if any(char.isdigit() for char in url):
        return "Digits are not allowed, we really like DNS!"

    try:
        response = requests.get(url, timeout=5)
        return response.text
    except Exception as e:
        return f"Request failed: {str(e)}"


if __name__ == "__main__":
    app.run(host="127.0.0.1", port=5000)
```

Found the app configuration file! Contains admin creds

![](attachments/Pasted%20image%2020260304124304.png)
Logged in and saw what looked like a test case for SSRF


>Testing SSRF

Didn't really get a way ahead with burp suite, so i decided to test directly.
Typed in http://127.0.0.1.nova.thm
and got a weird response
![](attachments/Pasted%20image%2020260304125123.png)

Figured i should get a way by using parameters like home, internal, localhost e.t.c
Got into the python sandbox page, but observed sumn, in the url after sending a url...
![](attachments/Pasted%20image%2020260304125632.png)
The application fetches url from the internal.nova.thm and executes it as code...so we have to find a possible 'url' what can be executed to help us read the flag in the application, 'flag.txt'

![](attachments/Pasted%20image%2020260304130442.png)

Got responses like this...
After several payloads were sent...i got the flag with this one: http://nova.thm/admin/fetch?url=http://internal.nova.thm?code=next(open(%22flag.txt%22))

![](attachments/Pasted%20image%2020260304131039.png)

Bankai!! I hate CTF's!
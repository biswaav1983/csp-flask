# csp-flask

What is CSP?
Content Security Policy is a computer security standard introduced to prevent cross-site scripting, clickjacking, and other code injection attacks resulting from the execution of malicious content in the trusted web page context.

What does the content security policy do?
Content Security Policy (CSP) is an additional layer of security that helps detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks. These attacks are used for everything from data theft to site defacement to malware distribution.

What is a good content security policy?
A strict content security policy is based on nonces or hashes. Using a strict CSP prevents hackers from exploiting HTML injection flaws, forcing the browser to execute the malicious script. The policy is especially effective against classical stored, reflected, and various DOM XSS attacks.

What is the default CSP?
The default-src Content Security Policy (CSP) directive allows you to specify the default or fallback resources that can be loaded (or fetched) on the page (such as script-src or style-src, etc.).

What are the three types of security policies?
Three types of security policies in common use are program policies, issue-specific policies, and system-specific policies.

Note: For the best possible experience, run these labs on Mozilla Firefox.

Basics
Step 1: Go to ReportURI - Generate

Step 2: Enter github.com in the form and click on Import

You can see the Content Security Policy that applies to github.com

Starting the CSP Application.
Step 1: Navigate to directory where we have csp application
cd /root/csp-flask
Step 2: Build the CSP application
docker build -t csp-flask -f Dockerfile .
Step 3: Run the CSP application
docker run -d -p 8443:80 csp-flask:latest
The CSP application can be accessed at the following URL:
echo http://$(serverip):8443
Concept 1 - Output Encoding Essentials
Block execution of Client-side content with CSP
Note: For the best possible experience, run these labs on Mozilla Firefox.

Step 1: Go to CSP application
echo http://$(serverip):8443
Step 2: Open the Web Console with Ctrl + Shift + I or F12 on Windows and Linux, or Command + Option + I on MacOS.

Step 2: In the form, under the heading Content-Security-Policy Header paste the copied CSP into CSP String field.

"default-src 'self'; script-src 'self'; img-src 'self'"
Step 3: In the payload section, enter: <img src="https://media2.giphy.com/media/1xl4TmDazTrk6ADuyi/giphy.gif" /> value.
<img src="https://media2.giphy.com/media/1xl4TmDazTrk6ADuyi/giphy.gif" />
Step 4: Click on the button Raw with CSP

Step 5: Run the same lab again, with this CSP String "default-src 'self'; script-src 'self'" and payload <img src="https://media2.giphy.com/media/1xl4TmDazTrk6ADuyi/giphy.gif" />

"default-src 'self'; script-src 'self'"
<img src="https://media2.giphy.com/media/1xl4TmDazTrk6ADuyi/giphy.gif" />
Step 6: Run the same lab with "default-src 'self'; script-src 'self'; img-src 'self'; frame-src 'none'" and payload <iframe src="https://www.appsecengineer.com">
"default-src 'self'; script-src 'self'; img-src 'self'; frame-src 'none'"
<iframe src="https://www.appsecengineer.com">
Analyzing the code
def raw_with_csp():
    if request.method == 'POST':
        if 'raw_payload' in request.form and 'csp_payload' in request.form:
            resp = make_response(render_template('results.html', payload=request.form['raw_payload']))
            resp.headers.set('Content-Security-Policy', request.form['csp_payload'])
            return resp
        else:
            return render_template('err.html', err={'error': "Invalid Data", "message": "Payload not in message"})
References
https://github.com/we45/csp-flask/blob/master/app/main.py
Bypass CSP
Note: For the best possible experience, run these labs on Mozilla Firefox.

Step 1: Navigate to the CSP application
echo http://$(serverip):8443Copy
Step 2: In the CSP Bypass Section Enter: https://ase-csp.sfo3.digitaloceanspaces.com/index.js
https://ase-csp.sfo3.digitaloceanspaces.com/index.jsCopy
Step 3: Click on the submit button

Analyzing the code

def csp_bypass():
    if request.method == 'POST':
        if 'raw_payload' in request.form:
            resp = make_response(render_template('bypass_1.html', payload=request.form['raw_payload']))
            resp.headers.set('Content-Security-Policy', "default-src 'none'; script-src 'self' *.digitaloceanspaces.com")
            return resp
        else:
            return render_template('err.html', err={'error': "Invalid Data", "message": "Payload not in message"})
Step 4: Repeat the same thing, except this time, try using https://ase-csp.sfo3.digitaloceanspaces.com/mal.js in the CSP Bypass Section
https://ase-csp.sfo3.digitaloceanspaces.com/mal.js
Step 5: Click on the submit button
References
https://github.com/we45/csp-flask/blob/master/app/main.py
Concept 2 - Hashes with CSP
CSP Hashes
Note: For the best possible experience, run these labs on Mozilla Firefox.

Step 1: Go to CSP application
echo http://$(serverip):8443
Step 2: In the CSP Hashes section, enter: alert('this is a xss attack ' + document.domain)
alert('this is a xss attack ' + document.domain)
Step 3: Click on the submit button

Step 4: Now, try and enter document.write("<h3>This is genuine JS</h3>"); in the CSP Hashes section

document.write("<h3>This is genuine JS</h3>");
Step 5: Click on the submit button

Analyzing the code

def csp_hash():
    if request.method == 'POST':
        if 'raw_payload' in request.form:
            resp = make_response(render_template('bypass.html', payload=request.form['raw_payload']))
            static_js = requests.get('https://ase-csp.sfo3.digitaloceanspaces.com/index.js').content.strip()
            csp_hash = b64encode(sha256(static_js).digest()).decode()
            resp.headers.set('Content-Security-Policy',
                             "default-src 'self'; script-src 'sha256-{}'".format(csp_hash))
            return resp
        else:
            return render_template('err.html', err={'error': "Invalid Data", "message": "Payload not in message"})
References
https://github.com/we45/csp-flask/blob/master/app/main.py
Concept 3 - CSP Nonces
CSP Nonce
Note: For the best possible experience, run these labs on Mozilla Firefox.

Step 1: Go to CSP application
echo http://$(serverip):8443
Step 2: In the CSP Nonce section enter: alert('xss')
alert('xss')
Step 3: Click on the submit button

Step 4: Right click and View Page Source

Wait for instructor

def csp_nonce():
    if request.method == 'POST':
        if 'raw_payload' in request.form:
            static_js = requests.get('https://ase-csp.sfo3.digitaloceanspaces.com/index.js').content.strip()
            csp_nonce = uuid4().hex
            resp = make_response(
                render_template('nonce.html', payload=request.form['raw_payload'], nonce=csp_nonce,
                                script=static_js.decode()))
            resp.headers.set('Content-Security-Policy',
                             "default-src 'self'; script-src 'nonce-{}'".format(csp_nonce))
            return resp
        else:
            return render_template('err.html', err={'error': "Invalid Data", "message": "Payload not in message"}
References
https://github.com/we45/csp-flask/blob/master/app/main.py

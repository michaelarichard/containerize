# Homework 1: Containerize

## Basic Use

```
docker-compose build && docker-compose up
```
Browse to https://localhost or `curl -k https://localhost`


## Development Notes

For local development, use the above instructions to run a local docker instance.
Edit src/app/server.py and save/refresh to see changes.  

If this needs to be deployed to a shared environment a helm chart + skaffold.yaml can be added to allow faster iteration and deployment of changes/settings depending on environmental needs.

## Nginx notes

Nginx config files are separated into several parts for flexibility. 
and may need to be adjusted to allow for scaling needs. 

Depending on deployment needs some settings can be baked into an nginx container using a Dockerfile, but as the dns names and ssl configuration may change depending on the environment, I opted to use docker-compose values instead.

upstream app_servers block can be edited to include additional servers and backup targets depending on needs.

I did not spend time to configure run nginx under a non-root user, as I am short on time this week and assume this will be handled by a real load balancer. I assume this reverse proxy is only for local development purposes, but if that's required, it can be done. Production environments should have a true ingress or lb configured instead.


## SSL/TLS
See nginx/ssl.conf for certificate file mapping and cipher configuration.

Currently, SSL ciphers are configured to a very high setting to remove warnings during the validate.sh script. 
This may cause compatibility issues with certain browsers.
For compatibility with IE8 or IE6, see the commented ssl_cipher line in nginx/ssl.conf. 

ssl_dhparam can be configured to strengthen key exchange after generating a prime using `openssl dhparam -out dhparams.pem 4096`
I opted to forgo this process as it can take awhile to generate and may cause performance issues. 

For additional context, see https://medium.com/@davetempleton/tls-configuration-cipher-suites-and-protocols-a01ee7005778
also https://wiki.mozilla.org/Security/Server_Side_TLS


## validate.sh output
```
➜  containerize git:(master) ✗ sh validate.sh
No stopped containers
Step 1/14 : FROM python:3.7.2-alpine
 ---> bb1ccaa5880c
Step 2/14 : RUN pip install --upgrade pip
 ---> Using cache
 ---> 63e6423d3139
Step 3/14 : RUN adduser -D worker
 ---> Using cache
 ---> df81a549c1e2
Step 4/14 : USER worker
 ---> Using cache
 ---> c306812af8c3
Step 5/14 : WORKDIR /home/worker
 ---> Using cache
 ---> cea6dff76957
Step 6/14 : COPY --chown=worker:worker src/requirements.txt requirements.txt
 ---> 056564d41a39
Step 7/14 : RUN pip install --user -r requirements.txt
 ---> Running in 9c0ff9b75587
Collecting Flask==1.1.1
  Downloading Flask-1.1.1-py2.py3-none-any.whl (94 kB)
Collecting gunicorn==19.9.0
  Downloading gunicorn-19.9.0-py2.py3-none-any.whl (112 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.0-py2.py3-none-any.whl (298 kB)
Collecting Jinja2>=2.10.1
  Downloading Jinja2-2.11.1-py2.py3-none-any.whl (126 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting click>=5.1
  Downloading click-7.1.1-py2.py3-none-any.whl (82 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1.tar.gz (19 kB)
Building wheels for collected packages: MarkupSafe
  Building wheel for MarkupSafe (setup.py): started
  Building wheel for MarkupSafe (setup.py): finished with status 'done'
  Created wheel for MarkupSafe: filename=MarkupSafe-1.1.1-py3-none-any.whl size=12629 sha256=1afa929c62aee753a6c2a8c9453224f9f22b124132c50776394afcae52a3a907
  Stored in directory: /home/worker/.cache/pip/wheels/b9/d9/ae/63bf9056b0a22b13ade9f6b9e08187c1bb71c47ef21a8c9924
Successfully built MarkupSafe
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, click, Flask, gunicorn
  WARNING: The script flask is installed in '/home/worker/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The scripts gunicorn and gunicorn_paster are installed in '/home/worker/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed Flask-1.1.1 Jinja2-2.11.1 MarkupSafe-1.1.1 Werkzeug-1.0.0 click-7.1.1 gunicorn-19.9.0 itsdangerous-1.1.0
Removing intermediate container 9c0ff9b75587
 ---> 4d4db0d8c5f7
Step 8/14 : ENV PATH="/home/worker/.local/bin:${PATH}"
 ---> Running in 91df15145b27
Removing intermediate container 91df15145b27
 ---> 993480b7f45c
Step 9/14 : COPY --chown=worker:worker src /app
 ---> 7262b72a3959
Step 10/14 : ENV FLASK_RUN_PORT 8000
 ---> Running in 4f4b0c0ca227
Removing intermediate container 4f4b0c0ca227
 ---> f893617aa6e4
Step 11/14 : ENV FLASK_APP /app/server.py
 ---> Running in 2c8bad1166d8
Removing intermediate container 2c8bad1166d8
 ---> a46e077f3fc3
Step 12/14 : ENV FLASK_ENV production
 ---> Running in b5718fe6c3e2
Removing intermediate container b5718fe6c3e2
 ---> 918615744e7f
Step 13/14 : EXPOSE 8000
 ---> Running in 3a3bb7423873
Removing intermediate container 3a3bb7423873
 ---> c62f32da798d
Step 14/14 : CMD ["gunicorn", "--bind", "0.0.0.0:8000", "wsgi", "--chdir", "/app", "--workers=2"]
 ---> Running in a9d4c17746ce
Removing intermediate container a9d4c17746ce
 ---> ab650576bef4
Successfully built ab650576bef4
Successfully tagged containerize_app:latest
1.17.9-alpine: Pulling from library/nginx
Digest: sha256:abe5ce652eb78d9c793df34453fddde12bb4d93d9fbf2c363d0992726e4d2cad
Status: Downloaded newer image for nginx:1.17.9-alpine
------------------------------------------------------------------------
| Testing SSL/TLS settings...                                          |
------------------------------------------------------------------------

 Start 2020-03-27 03:17:01        -->> 127.0.0.1:443 (localhost) <<--

 A record via:           /etc/hosts 
 rDNS (127.0.0.1):       localhost.
 Service detected:       HTTP


 Testing protocols via sockets except NPN+ALPN 

 SSLv2      not offered (OK)
 SSLv3      not offered (OK)
 TLS 1      not offered
 TLS 1.1    not offered
 TLS 1.2    offered (OK)
 TLS 1.3    offered (OK): final
 NPN/SPDY   http/1.1 (advertised)
 ALPN/HTTP2 http/1.1 (offered)

 Testing cipher categories 

 NULL ciphers (no encryption)                  not offered (OK)
 Anonymous NULL Ciphers (no authentication)    not offered (OK)
 Export ciphers (w/o ADH+NULL)                 not offered (OK)
 LOW: 64 Bit + DES, RC[2,4] (w/o export)       not offered (OK)
 Triple DES Ciphers / IDEA                     not offered
 Obsolete: SEED + 128+256 Bit CBC cipher       not offered
 Strong encryption (AEAD ciphers)              offered (OK)


 Testing HTTP header response @ "/" 

 HTTP Status Code             200 OK
 HTTP clock skew              0 sec from localtime
 Strict Transport Security    365 days=31536000 s, includeSubDomains
 Public Key Pinning           --
 Server banner                nginx/1.17.9
 Application banner           --
 Cookie(s)                    (none issued at "/")
 Security headers             X-Frame-Options SAMEORIGIN
                              X-XSS-Protection 1; mode=block
                              X-Content-Type-Options nosniff
 Reverse Proxy banner         --


 Done 2020-03-27 03:17:16 [  17s] -->> 127.0.0.1:443 (localhost) <<--


------------------------------------------------------------------------
| Testing HTTP header and body content...                              |
------------------------------------------------------------------------
Pass: status code is 200
Pass: X-Forwarded-For is present and not 'None'
Pass: X-Real-IP is present and not 'None'
Pass: X-Forwarded-Proto is present and not 'None'
Pass: found "It's easier to ask forgiveness than it is to get permission." in response
------------------------------------------------------------------------
| Testing hot reload for local development...                          |
------------------------------------------------------------------------
Pass: found "It's harder to ask forgiveness than it is to get permission." in response
------------------------------------------------------------------------
| Docker image sizes:                                                  |
------------------------------------------------------------------------
containerize_app:latest 104MB
Going to remove containerize_nginx_1, containerize_app_1
```
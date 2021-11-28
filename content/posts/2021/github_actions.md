---
title: "How to set up automated deployment using github actions"
date: 2021-11-28T18:24:00+02:00
draft: false
tags: ['server', 'devops', 'github_actions']
featured_image: https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fme-dutour-mathieu.gallerycdn.vsassets.io%2Fextensions%2Fme-dutour-mathieu%2Fvscode-github-actions%2F3.0.0%2F1588593426297%2FMicrosoft.VisualStudio.Services.Icons.Default&f=1&nofb=1
description: You can replace your docker with a 
---

I don't like managing servers.
This is a well-known fact.
My way around it was to set up all my apps using docker, and that made me happy.

After my small server ran out of space with only 5 web apps running, I started changing my mind.
On top of this, I used GitHub actions with azure, and loved it.

So this is how I moved my dotnet webapp from docker to a Linux service automatically deployed by github actions

To better explain this, I am going to use pitaco, a comment box I have created for studying purposes: https://github.com/tiagodll/pitaco


# 1. Setup the app as a Linux service

clone your app and publish it in your home folder
```bash
git clone git@github.com:tiagodll/pitaco.git
cd pitaco
dotnet publish src/Pitaco.Server/ -c Release -o ./publish --self-contained --runtime linux-x64
cp –R ./publish /full/path/to/pitaco
```


then create an ini file to set it up as a service

```ini
[Unit]
Description=Pitaco kestrel service

[Service]
WorkingDirectory=/full/path/to/pitaco
ExecStart=/full/path/to/pitaco/Pitaco.Server
SyslogIdentifier=Pitaco
Restart=always
User=your-user
RestartSec=5
# copied from dotnet documentation at
# https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-3.1#code-try-7
KillSignal=SIGINT
Environment="ASPNETCORE_ENVIRONMENT=Production"
Environment="ASPNETCORE_URLS=http://+:6000"
Environment="CONNECTION_STRING=Data Source=/path/to/pitaco.db;Pooling=True"
Environment="DOTNET_PRINT_TELEMETRY_MESSAGE=false"

[Install]
WantedBy=multi-user.target
```
then install the service
```bash
sudo cp pitaco.service /etc/systemd/system/pitaco.service
sudo systemctl daemon-reload
sudo systemctl start pitaco
```

# 2. set up a reverse proxy if you don't already have it

prerequisite: SSL certificates (you can use lets-encrypt)

remember to use the same port set in the service

```nginx
server {
    if ($host = pitaco.dalligna.com) {
        return 301 https://$host$request_uri;
    }
    listen 80;
    server_name pitaco.dalligna.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen              443 ssl;
    server_name         pitaco.dalligna.com;
    ssl_certificate     /etc/letsencrypt/live/pitaco.dalligna.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/pitaco.dalligna.com/privkey.pem; 

    location / {
        proxy_pass  http://127.0.0.1:6000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
```

# 3. set up github actions

Create some secrets:
- SSH_PRIVATE_KEY: your private key
- REMOTE_HOST: your server pi or domain
- REMOTE_USER: the user that your service is going to run as
- REMOTE_TARGET: the folder where your app is deployed

just click in actions, and create your yaml based on this:

```yml
name: .NET

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Set up .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '6.0.x'

      - name: Build with dotnet
        run: dotnet build src/Pitaco.Server/ --configuration Release

      - name: dotnet publish
        run: dotnet publish src/Pitaco.Server/ -c Release -o ./publish --self-contained --runtime linux-x64
      
      - name: copy published files
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SOURCE: "publish/"
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: ${{ secrets.REMOTE_TARGET }}
```

and that is all.

Now your app will be automatically deployed on every commit to main

you are welcome :smile:
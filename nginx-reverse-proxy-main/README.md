# nginx-reverse-proxy

![Untitled](https://github.com/user-attachments/assets/45eaa847-6ebb-467d-bf3f-e7ddf0dc7bd3)


This is a repository which shows how to setup nginx reverse proxy to route requests to different servers.

We will use docker containers to demonstrate this.

It consists of three nginx servers.

## Steps to follow

- Run docker-compose up

```
docker-compose up
```

- Go to localhost:8080/

- This delivers the main app

- Go to localhost:8080/app/

- Reload the app and it redirects to app1 and app2

## Read about it here

- [Read on codeBlog](http://codeblog.free.nf/2025/03/06/nginx-practical-guide/)

  

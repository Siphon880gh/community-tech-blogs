# Migration Guide

To make the app user friendly on a mixed VPS / Dedicated Server (mixed as in it can support php, nodejs, AND python) which helps reduce costs, reverse proxy is used to pass a friendly url's request to an internal port.

Database: Adjust the DB_NAME, DB_USER, DB_PW, DB_PORT at the .env file for authenticating and accessing your MySQL database. The DB_PORT is usually the same port across all your MySQL apps on this same server.

Ports: Assign a unique port for your Express app at .env's PORT variable. This needs to be unique from all your other Express apps on this same server.

Base URL - Here's how you replace the baseURL for this app:
- Run grep for `/app/community-tech-blogs/` and replace with your new baseURL. Or you may use sed to match and replace.

Envs:
- ./.env

Reverse Proxies: 
- `/app/community-tech-blogs[/]*` proxy passes to `127.0.0.1:PORT` where PORT is the unique Express port.

Eg.
```
  # Reverse proxy
  location ~ /app/community-tech-blogs[/]* {
    rewrite ^/app/community-tech-blogs[/]*(.*)$ /$1 break;
    proxy_pass http://127.0.0.1:3001;
    proxy_read_timeout 300s;   # Adjust as needed
    proxy_connect_timeout 300s; # Adjust as needed
    proxy_send_timeout 300s;   # Adjust as needed
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Enable CORS
    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Authorization' always;
    
    # Handle OPTIONS (preflight) requests
    if ($request_method = OPTIONS) {
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
      add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Authorization';
      add_header 'Access-Control-Max-Age' 1728000;
      return 204;
    }
  }
```

Start script at remote server:
- From the app root folder: `npm run start`
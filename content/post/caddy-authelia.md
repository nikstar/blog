---
title: Private server with Caddy and Authelia
date: 2024-02-18T18:44:08Z
draft: true
toc: true
---

I wanted to password-protect some parts of my personal website. In the process I switched from Nginx to Caddy and implemented sign in using Authelia. This transition was fairly straightforward and I wish I’d done it sooner.

  

## Caddy

Caddy is a modern web server written in Go. I switched to it from Nginx because it has shorter configs and simpler integration with Authelia. (Plus it’s new and shiny.) Latest version of Caddy can be downloaded [from Github](https://github.com/caddyserver/caddy/releases) and installed with

```bash
sudo dpkg -i caddy*.deb
```

Now is a good time to stop nginx server to prevent conflicts:

```bash
sudo systemctl stop nginx
sudo systemctl disable nginx
```

Unlike Nginx, the entire Caddy config fits in one file — `/etc/caddy/Caddyfile`. Config can be quickly reloaded.

```bash
sudo systemctl reload caddy
```

[Official documentation](https://caddyserver.com/docs/) is nice but here are some highlights of what I personally needed. 


### Static file server

Each domain name is defined as a top level block:

  

```caddyfile
example.com {
	root * /path/to/files
	file_server
}
```

This is pretty much everything you need to serve static files. It serves `index.html` for `/` and correctly handles trailing slashes.

One thing I configured additionally was a custom 404 page:

```caddyfile
example.com {
	root * /path/to/files
	file_server
	
	handle_errors {
		@404 {
			expression {http.error.status_code} == 404
		}
		rewrite @404 /404.html
		file_server
	}
}
```

The block starting with `@` defines a selector and following lines handle it by serving 404.html.

### HTTPS

Caddy automatically issues required SSL certificates with Let’s Encrypt and requires literally zero configuration. The code above already supports HTTPS and redirects HTTP requests.

For reference, equivalent setup for Nginx with certificates managed by Certbot looked like this:

```nginx
server {
	server_name example.com;
	
	location / {
		if ($request_uri ~ ^/(.*)\.html$) {
			return 302 /$1;
		}
		
		root /path/to/files;
		try_files $uri $uri.html $uri/ =404;
		
		error_page 404 /404.html;
	}
  
	listen [::]:443 ssl; # managed by Certbot
	listen 443 ssl; # managed by Certbot
	ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
	ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
	include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
	ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
	  
server {
	if ($host = example.com) {
		return 301 https://$host$request_uri;
	} # managed by Certbot

	server_name example.com;

	listen 80;
	listen [::]:80;
	return 404; # managed by Certbot
}

```

An absolute win for Caddy in my book, and one less dependency.

### Reverse proxy

Reverse proxy forwards requests to some service (in this example running on the same server, port 3001):


```caddyfile
example.com {
	reverse_proxy /service/* http://localhost:3001
	
	root * /path/to/files
	file_server
	...
}
```

  

Note that directives are evaluated in order, so more selective `reverse_proxy /service/*` should go before catch-all `file_server`. 

The above example won’t handle `example.com/service` without a trailing slash. Use a selector to support it:

```caddyfile

example.com {
	@service path /service /service/*
	handle @service {
		reverse_proxy http://localhost:3001
	}
	...
}

```


Once again, Caddy has sensible defaults and automatically sets `X-Forwarded-*` headers.

### Access logs
Caddy server logs can be viewed with `journalctl`:

```bash
sudo journalctl -u caddy | tail
```

They include a lot of random stuff so it's best to keep a separate access log.

```caddyfile
example.com {
	log {
		output file "/var/log/caddy/example.com-access.log" {
			roll_size "1MiB"
			roll_keep 1
		}
	}
}
```

Caddy really wants you to use structured logs so it's best not to fight it but instead parse them with `jq`:

```bash
sudo tail -f /var/log/caddy/example.com-access.log | jq -r '[.request.client_ip,.request.method,.request.uri,.status] | @tsv'
```
This will show recent accesses as a nice table:
```tsv
...
1.2.3.4      GET     /index.xml      200
1.2.3.4      GET     /favicon.ico    404
1.2.3.4      GET     /               304
```


## Authelia

Time to add password authentication to our service. Authelia is an authentication service also written in Go. Download it [from GitHub](https://github.com/authelia/authelia/releases).

I won’t go through every config option (`/etc/authelia/configuration.yml`) because everything is extensively commented. I just went through it slowly picking the easiest option and it worked out okay. Once it launches successfully, time to make changes to Caddy config. We’ll put `/service` behind login:

  

```caddyfile
example.com {
	@authelia path /login /login/*
	handle @authelia {
		reverse_proxy http://localhost:9091
	}

	@service path /service /service/*
	handle @service {
		
		forward_auth http://localhost:9091 {
			uri /api/verify?rd=https://example.com/login
			copy_headers Remote-User Remote-Groups Remote-Name Remote-Email
		}
		
		reverse_proxy http://localhost:3001
	}
	...
}

```

  

Let’s unpack this.

  

The first endpoint is the actual login page. Clients can go there directly to sign in or out but they usually won't have to.

When a client tries to access protected `/service`, `forward_auth` first makes a request to Authelia's `/api/verify` endpoint. 
	
+ If auth cookie is set and valid, Authelia returns status code `200` and some headers, and Caddy proceeds to `reverse_proxy`. 
+ If the client is not authorized, Authelia will return a redirect (code `301`) to login page and protected resource won’t be accessed. Once the client is logged in they will be redirected back and authorization will be attempted again (successfully).

  

### Reusability

To protect multiple endpoints, pull out forward_auth code into a snippet and use it for each service.

```caddyfile

(protected_endpoint) {
	forward_auth http://localhost:9091 {
		 uri /api/verify?rd=https://example.com/login
		 copy_headers Remote-User Remote-Groups Remote-Name Remote-Email
	 }
}
 
example.com {
	...
	@service1 path /service1 /service1/*
	handle @service1 {
		import protected_endpoint
		reverse_proxy http://localhost:3001
	} 
	
	@service2 path /service2 /service2/*
	handle @service2 {
		import protected_endpoint
		reverse_proxy http://localhost:5555
	}
	...
}
```

Now adding a new service will take just 5 lines.

<br><br>

And just like that we have a fresh new web server with password-protected services.

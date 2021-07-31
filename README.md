# Running a single containerized Plumber API on DigitalOcean with Docker Compose, nginx, and Let's Encrypt <!-- omit in toc -->

> tl;dr: This repository contains a barebones example plumber API that lives in a Docker container that *also* works with an nginx Docker container that *also* works with a Let's Encrypt HTTPS certbot container that *also* all gets built and orchestrated with Docker Compose. It's basically the result of following [this DigitalOcean guide](https://www.digitalocean.com/community/tutorials/how-to-secure-a-containerized-node-js-application-with-nginx-let-s-encrypt-and-docker-compose), but substituting plumber for the example Node.js app they use.

---

- [Why plumber?](#why-plumber)
- [Deploying plumber APIs](#deploying-plumber-apis)
- [The holy grail: containers for everything *and* HTTPS!](#the-holy-grail-containers-for-everything-and-https)

## Why plumber?

Plumber is an incredible R package that makes it really easy to create APIs within R. APIs are great for data analysis and machine learning and anything computationally intensive because you can (1) offload those computations to an external server and (2) make it easy for anyone else to access or interact with those results. Some have even said that [plumber is the key to putting R in production](https://twitter.com/JosiahParry/status/1415707758175965195), and that although it's really tempting to put all sorts of complex logic and calculations in things like Shiny apps and flexdashboards, [it's better to offload that to an API](https://twitter.com/JosiahParry/status/1415708339330338817).

There are a ton of resources out there for learning how to use plumber:

- [The plumber quickstart guide](https://www.rplumber.io/articles/quickstart.html) (plumber's documentation in general is phenomenal)
- A two-part series by Heather and Jacqueline Nolis: ["R can API and So Can You!"](https://medium.com/tmobile-tech/r-can-api-c184951a24a3) (Part 1); ["Using Docker to Deploy an R Plumber API"](https://medium.com/tmobile-tech/using-docker-to-deploy-an-r-plumber-api-863ccf91516d) (Part 2)
- [REST APIs and Plumber](https://rviews.rstudio.com/2018/07/23/rest-apis-and-plumber/)
- [REST APIs with R and Plumber](https://appsilon.com/r-rest-api/)
- [How to make your machine learning model available as an API with the plumber package](https://shirinsplayground.netlify.app/2018/01/plumber/)

## Deploying plumber APIs

Once you have a working API, you'll want to deploy it somewhere so it's accessible outside of your computer. This is where it gets tricky!

The plumber documentation has a [section on hosting](https://www.rplumber.io/articles/hosting.html) which is really helpful and which outlines a few different ways to make these public:

- [RStudio Connect](https://www.rstudio.com/products/connect/): the easiest method, but holy crap it's expensive
- Self-hosted on a server like DigitalOcean: this is cheap, but it requires configuring a Linux server that works with R and a web server
- Self-hosted within a Docker container: this is also cheap, and it makes configuration super simple—you can create a Docker container that has [R and the tidyverse pre-installed](https://hub.docker.com/r/rocker/tidyverse) so that it's really quick to create a working computer without much additional configuration.

In my own use-case, I'm creating a nerdy personal API that collects data from all over the internet ([Airtable databases](https://airtable.com/) I use for tracking research, Google Sheets I use for tracking research and teaching and other side projects, my Fitbit data, and so on). I can have all my data accessible from a single API—I can make a POST request to `apiaddress.com/health` and get JSON data of a subset of my Fitbit data; `apiaddress.com/pipeline` and get JSON data with details of the projects in my research pipeline; etc.

My goal with deployment is to make it as easy (and as cheap) as possible to stick my API online with as little manual configuration as possible. Ideally, I want an ephemeral setup with Docker containers for the plumber API and for the web server infrastructure so that I can stick it on a cheap Digital Ocean droplet and have it Just Work™.

This is hard though! The [plumber hosting documentation](https://www.rplumber.io/articles/hosting.html) is great and shows a few examples of similar setups:

- [Hosting an API directly on Digital Ocean](https://www.rplumber.io/articles/hosting.html#digitalocean-1), sans Docker—easy, but requires server setup and it's sometimes tricky to install R on low-memory servers
- [Hosting a single API inside Docker](https://www.rplumber.io/articles/hosting.html#docker-basic-)—easy, but is just one container, which means you still need to configure a web server
- [Hosting multiple APIs](https://www.rplumber.io/articles/hosting.html#multiple-plumber-applications-1) (or a [single API multiple times](https://www.rplumber.io/articles/hosting.html#multiple-applications-on-one-port-1) for [load balancing](https://www.rplumber.io/articles/hosting.html#load-balancing-1) purposes) insider Docker containers that *also* work with nginx web server Docker containers, all orchestrated with [Docker Compose](https://docs.docker.com/compose/)

That last option seems almost ideal. I want to host a single API inside a Docker that is served through nginx (also inside a Docker container), that's all magically orchestrated with Docker Compose. But there's no easy example of that basic setup.

Additionally, none of the options in the plumber documentation show how to get this all working with HTTPS (which is important if you have POST endpoints). Getting HTTPS enabled is relatively easy with Let's Encrypt, which provides free SSL certificates, but getting that working with nginx inside a Docker container seems really really hard.

## The holy grail: containers for everything *and* HTTPS!

After a ton of googling and hours of failed attempts, I finally found that DigitalOcean has a guide for hosting a simple single Node.js app in a Docker container, which is passed through an nginx Docker container, which has Let's Encrypt-based HTTPS, all orchestrated with Docker Compose. If you swap out the Node.js app for a plumber app, that's basically my holy grail!

But, [DigitalOcean has a guide](https://www.digitalocean.com/community/tutorials/how-to-secure-a-containerized-node-js-application-with-nginx-let-s-encrypt-and-docker-compose) for hosting a single Node.js app through nginx with Let's Encrypt-based HTTPS. That's almost the holy grail!

So I followed their very detailed and pedagogically phenomenal instructions, substituting a basic plumber app for the Node app they refer to, and it works! It's so magical and neat. Using just a `docker-compose.yml` file, a `Dockerfile`, and a few other configuration files, I can fire up a basic DigitalOcean server, do some super minor configuration (which I could probably automate with [their User Data system](https://docs.digitalocean.com/products/droplets/how-to/provide-user-data/)), run `docker-compose up -d`, and have a working API in a matter of minutes.

[Follow DigitalOcean's guide here](https://www.digitalocean.com/community/tutorials/how-to-secure-a-containerized-node-js-application-with-nginx-let-s-encrypt-and-docker-compose) and you can do the same thing.

Mostly as a reference for future-me (but also for anyone else), this repository contains a barebones API that's based on the DigitalOcean guide. Really the only things that needed to change to make it work for plumber rather than Node are (1) the name of the application (they called their container instance "nodejs"; here it's called "plumber"), and (2) the port (they use 8080, which I guess is normal in Node world(?), while most plumber examples I've seen use 8000, so I used 8000 throughout all the configuration files).

(As an extra neat bonus, if you use [Visual Studio Code](https://code.visualstudio.com/) + [the SSH extension](https://code.visualstudio.com/docs/remote/ssh) + [the Docker extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker), you can edit files remotely *and* manage Docker containers and Docker Compose remotely, all from within Visual Studio Code, which is exceptionally magical.)

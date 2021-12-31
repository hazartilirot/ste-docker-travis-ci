# Notes to the course

Launch in a detached mode

`docker-compose up -d`

Stop all running containers at once

`docker-compose down`

Re-build images, but use cache unless there is a change in docker-compose.yml

`docker-compose build`

Re-build images from scratch omitting forcefully cache,

`docker-compose build --no-cache`

Restart policies in docker-compose file for services:
- restart: "no" (the value must be wrapped in quotes. Otherwise, yml interprets it as boolean value)
- restart: always (if the container stops for any reason, restart it)
- restart: on-failure (self-explanatory, restart container if a container stops with a code error)
- restart: unless-stopped (restart always unless a maintainer forcibly stops it)

Show running processes. Mind the command should be run in the project's directory relating to the docker-compose.yml Otherwise it'll throw an error.

`docker-compose ps`

A docker file might consist an additional extension .dev in its name like 
Dockerfile.dev in a project. To run the file we have to specified it by a 
switch. The feature is used to differentiate a production build from a 
development one

`docker build -f Dockerfile.dev .`

In order to expose a port to a local machine we need to use '-p' a 
port-mapper switch in our command. The left side is a local side, the right 
is a container

`docker run -p 3000:3000 <IMAGE ID/NAME>`

If we're working in our IDE with a running docker's instance of our project 
in development and want to reflect changes in our code in the container 
(like a watch mode), we need to make use of volumes:

`docker run -p 3000:3000 -v $(pwd):/app <IMAGE ID/NAME>`

If we don't have node_modules installed in our project directory, or we want 
to skip, we need to add extra -v switch. Both approaches would work in 
Windows provided WSL is installed with bash. Otherwise, it'd be easier to 
create a docker-compose.yml file in which you specify volumes

`docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <IMAGE ID/NAME>`

Mind there might be an issue with reflecting a change in Windows. The actual 
change would be written into a file (mirrored) provided you want to see the 
change execute the command in a new terminal window:

`docker exec -it <CONTAINER ID> sh`

once you're in a home directory, see the changes in /app, for no reason 
React's server won't reload the changes.

The only solution found is to migrate the whole project to WSL and run 
it from there. `\\wsl$\Ubuntu-20.04\home\<USERNAME>\travis-ci`

`CHOKIDAR_USEPOLLING=true` - is a waste of time.

If we're going to execute a command within a container we want to run use 
the following line. Mind a subtle difference. If you execute it, it won't 
reflect changes as a hot reload (or live reload) does. 

`docker run -it <IMAGE ID> npm run test`

Therefore, run a container with `docker-compose up` command and then, when 
it's running, execute:

`docker exec -it <CONTAINER ID> npm run test`

It's a temporary solution provided you don't want to use a second service in 
**docker-compose.yml** file for running tests.

Another solution would be to use services:
````
  test:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
````
The downside of the approach would be inaccessibility of a test menu though 
it definitely reflects your changes in code by re-running tests. Mind we use 
**command** property to override **CMD** specified in **Dockerfile.dev**

For a production we specify a multistep process in which stage **as 
builder** signifies all we care of is just the result. The rest can be 
destroyed once the project has been built.

`FROM node:16.13.1-alpine as builder`

In the next stage our result is retrieved from the build stage and placed
into nginx directory:

`COPY --from=builder /app/build /usr/share/nginx/html`
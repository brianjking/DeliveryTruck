# Delivery Truck
### What will *you* deliver next?

Delivery Truck is a lightweight deployment tool that was developed as a solution to a problem that small and large developers face everyday -- whats the fastest way to deploy this app to my server. Indeed, you could deploy to Heroku or Azure with a few lines, but isn't it much cooler to run your own infrastructure? This is where Delivery Truck comes in.

## Quick Start
```
npm install deliverytruck -g
```

Let us imagine that your have created a genius  Meteor application and are ready to deploy.
You will certainly be updating it many times over and when you finally decide to push your code you'll have to go through the gruesome process of bundling, copying your files to the server, settings environment variables, etc. 
The problem grows even bigger if you have multiple servers, at which point you're simply require some automation.

With Delivery Truck, you can simply run:
```
truck deploy
```
To deploy to all of your servers and update your app.

## How Does It Work?
Delivery Truck relies on .truck.yml files for its configuration, one per deploy target, with quick.truck.yml being the default if your don't specify a target. These two lines are equivalent:
```
truck deploy
truck deploy quick
```
In fact, the quick.truck.yml can be auto generated (or any other *.truck.yml) with
```
truck init
```
or
```
truck init quick
```
or
```
truck init awesomesauce
```

The output of `truck init` is fixed (for now), and outputs the following:
```yaml
servers:
  - ssh://deploy@main.example.com
  - ssh://anotherdeploy@notmain.example.com:398
prepare:
  - meteor bundle bundle.tar.bz2
files:
  - bundle.tar.bz2
  - settings.json
deploy_env:
  - PORT: 4000
  - MONGO_URL: mongodb://localhost:27017/test
  - ROOT_URL: http://example.com/
  - METEOR_SETTINGS: "@settings.json"
deploy:
  - tar -zxf bundle.tar.bz2
  - cd bundle
  - cd programs/server
  - npm install
  - cd ../..
  - forever restart main.js
```

It's general enough to show the syntax of `.truck.yml` file, and at the same time a pretty good template for a Meteor project.
Lets go through this line by line.

### Servers Directive
```yaml
servers:
  - ssh://deploy@main.example.com
  - ssh://anotherdeploy@notmain.example.com:398
```
Here we set our deploy servers, the user that is used to connect to it, and the port.
Notice the ssh://, it is actually very important and without it Delivery Truck won't run at all (for forward compatibility).
The connection to these will be made using public-key authentication and no other method is supported -- for a good reason.
The location of the private key file can be specified in 3 ways:
* TRUCK_KEYFILE environment variable
* --keyFile command line argument
* keyFile block in the config file

If no keyfile is specified, Delivery Truck will use "~/.ssh/id_rsa".

You can also specify a passphrase (if you're using one) with:
* TRUCK_PASSPHRASE environment variable
* --pasphrase comand line argument

### Prepare Directive
```yaml
prepare:
  - meteor bundle bundle.tar.bz2
```

This block defines which commands to run before actually deploying anything, this block is run on your machine once per invocation of truck.
You should use this to produce any and all files that you will need for a specific deployment.

### Files Directive
```yaml
files:
  - bundle.tar.bz2
  - settings.json
```

Here we define which files to upload to *every* server. As you can see bundle.tar.bz2 was produced in the prepare step, while settings.json are simply a file in the project directory.
If we wanted to put the `bundle.tar.bz2` in a different directory (Remote $HOME is the default location) we can do so with `- bundle.tar.bz2: /usr/var/otherFile.tar.bz2`
Pretty neat!

### Deploy Directives
```yaml
deploy_env:
  - PORT: 4000
  - MONGO_URL: mongodb://localhost:27017/test
  - ROOT_URL: http://example.com/
  - METEOR_SETTINGS: "@settings.json"
```

This directive defines the environment in which your deploy script is run. Here's a neat feature, the `- METEOR_SETTINGS: "@settings.json"` line tells Delivery Truck to initialize METEOR_SETTINGS with the data from the file settings.json.
Neato!
And of course you can also defined a "prepare_env".

```yaml
deploy:
  - tar -zxf bundle.tar.bz2
  - cd bundle
  - cd programs/server
  - npm install
  - cd ../..
  - forever restart main.js
```
This is the final step of the process. This directive defines the deploy instructions that will be run on *every* server. In this example we simply unzip the bundle, install npm dependecies and restart the server (forever and node were setup with an illusive `truck deploy install`)

> Thats cool man, but uh, what if something fails along the way

If there is an error at any step of the way (e.g. file not found, non-zero exit code, auth error, etc.) Delivery Truck will terminate and give you an error message.
However, if you disable strict mode (with `--nostrict` or `strict: false`) Delivery Truck will move on to the next server in line and tell you about misbehaving servers in the end.

Roadmap (in order of importance):
* Per-server keyFiles
* .truck.yml repository
* More directives (e.g. `deploy_fail:`, `finalize:`)
* Better integration with DigitalOcean etc. (e.g. do://myInstance or azure://myInstance)
* ~~???~~
* ~~PROFIT!~~

# Challenge

Now that you've learned about basic Kubernetes functions, it's time to put them to use.

The goal of this challenge is to setup a Wordpress application running with a mysql database.

You can find the wordpress container documentation on Docker hub [here](https://hub.docker.com/_/wordpress).
However, this documentation is for Docker Swarm so it will need adapting to run on Minikube.

You may be on your own for this challenge but here are a few guidelines on how to approach this.

1. Start by isolating the mysql application
  1. Use environment variables to store the Mysql configuration
1. Add the wordpress container and connect it to the mysql instance
  1. Use environment variables to store the Mysql configuration
1. Setup some persistent volumes to keep your wordpress data if the mysql pod restarts
1. Use config maps to store the mysql information instead of using env variables
1. Use secrets to store more sensitive data (user and password)

## More challenges
- Load your database in a different namespace as your wordpress application
    - This would come in handy if you want to use your database for multiple applications
    - Note that you will need to restart your wordpress pod if you change the database configuration

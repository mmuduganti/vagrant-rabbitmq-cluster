

Vagrant script to create a cluster of RabbitMq nodes using Chef Solo.

The vagrant file is commented with placeholders for rabbitmq and erlang versions.

Adds a user for remote access - remote-guest/remote-guest.


Dependancies
------------

Ruby 1.9.2
Vagrant 1.6+

Install vagrant plugins
-----------------------

```
vagrant plugin install vagrant-cachier
vagrant plugin install vagrant-omnibus
vagrant plugin install vagrant-hostmanager

```

Install knife-solo and librarian-chef
-------------------------------------

```
cd vagrant-rabbitmq-cluster
bundle

```

Install chef cookbooks
----------------------

```
librarian-chef install
```

and finally

```
vagrant up

```



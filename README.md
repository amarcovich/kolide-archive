<img style="float:left;" height="244px" src="https://github.com/kolide/kolide/blob/master/kolide-banner.png?raw=true">

# Kolide

[![Build Status](http://komanda.io:8080/api/badges/mephux/kolide/status.svg)](http://komanda.io:8080/mephux/kolide)
[![Go Report Card](https://goreportcard.com/badge/github.com/mephux/kolide)](https://goreportcard.com/report/github.com/mephux/kolide)

  Kolide is an agentless osquery (https://osquery.io/) web interface and remote api server. Kolide uses the osquery
  remote apis to do ad-hoc distributed queries, osqueryd configurations and the collection and processing of scheduled queries (packs). Kolide was designed to be extremely portable (a single binary) and performant while keeping the codebase simple.

<img src="https://github.com/kolide/kolide/blob/master/kolide.gif?raw=true">

# What is osquery?

 > osquery allows you to easily ask questions about your Linux and OSX infrastructure. Whether your goal is intrusion  detection, infrastructure reliability, or compliance, osquery gives you the ability to empower and inform a broad set of organizations within your company." - via https://osquery.io/

# Announcing Kolide, Inc.

 Hi Kolide enthusiasts! Based on the tremendous response to Kolide we are excited to announce that the project has been transferred to a new company called [Kolide, Inc](https://www.kolide.co).

 Founded by key members of the core osquery team, Kolide Inc’s mission is to continue the development of the Kolide open source project with the goal of making it the _best_ way to manage any osquery fleet, big or small.

 With that mission in mind, we are hard at work with a rewrite of the project so we can ship many of the new features and improvements suggested by the community.

 Our goal is to get this first release out within a few months. Until then, **this version of Kolide will be in maintenance mode and will not receive any new features or significant updates**.

 Please continue to submit issues and support requests. We will respond to them as quickly as we can, and ensure ideas are captured for our next open source release.

 Watch this space for updates!

 Jason Meller<br>
 CEO of Kolide, Inc.

# Kolide Rest API & OSquery Remote API

  Kolide has a rest api that uses jwt (https://jwt.io/) for authentication. The only exception
  to this is the osquery remote api below.

  [osquery remote api](https://osquery.readthedocs.org/en/stable/deployment/remote/#remote-server-api):

  method | url | osquery configuration cli flag
  -------|-----|-------------------------------
  POST   | /api/v1/osquery/enroll | `--enroll_tls_endpoint`
  POST   | /api/v1/osquery/config | `--config_tls_endpoint`
  POST   | /api/v1/osquery/log    | `--logger_tls_endpoint`
  POST   | /api/v1/osquery/read   | `--distributed_tls_read_endpoint`
  POST   | /api/v1/osquery/write  | `--distributed_tls_write_endpoint`

# Building / Development

  1. To build Kolide from the source code yourself you need to have a working
  Go environment with [version 1.5 or greater installed](http://golang.org/doc/install).

  2. Get the code into your $GOPATH

  `GO15VENDOREXPERIMENT=1 go get github.com/kolide/kolide`

  (Note that errors referencing `Asset`, `AssetDir`, `AssetInfo` and `MustAsset` can be ignored for now. They will be fixed later by `go-bindata` in `make`.)

  or

  `git clone https://github.com/kolide/kolide.git`


  3. Install build/dev dependencies.

  `make deps`

  4. Build the code.

  `make`

  5. Start postgres and redis

    The easiest way to start writing code is to use docker/docker-compose.

    * `make up` will run docker-compose and bootstrap the deps
    * `make down` will spin down and remove all deps

## Certs/Openssl

  `make certs-remote cn=example.com` // change this to your domain

  This will drop a kolide.crt and kolide.key into ./tmp and copy them to /tmp

## Running Kolide

  ```
usage: kolide --config=CONFIG [<flags>]

osquery command and control

Flags:
      --help                     Show context-sensitive help (also try --help-long and --help-man).
      --debug                    Enable debug mode.
  -q, --quiet                    Remove all output logging
      --dev                      enable dev mode (serve assets from disk)
  -c, --config=CONFIG            configuration file
      --production               enable production mode
      --address=:8000            web server network address
      --enroll-secret=secret     osquery enroll secret
      --db-address=:5432         database network address
      --db-username=kolide       database username
      --db-password=secret       database password
      --db-database=kolide       database database
      --redis-address=:6379      redis network address
      --redis-protocol="tcp"     redis network protocol
      --redis-size=10            redis maximum number of idle connections
      --redis-password=secret    redis password
      --redis-secret-key=secret  redis secret key
      --version                  Show application version.
  ```

  The Kolide configuration file is required. Below is an example configuration file:

  ```toml
[session]
# Change This
key = "change-this-ahAJDKFH876A*DS&R*SADf187@#$#%^$%^11+0"
size = 10
network = "tcp"
address = "127.0.0.1:6379"
password = ""

[database]
# postgres or sqlite
type = "postgres"
address = "127.0.0.1:5432"
username = "kolide"
password = "kolide"
database = "kolide"
ssl = "disable"
cert = ""
key = ""

[server]
# enroll_secret = "kolidedev"
enroll_secret = ""
query_timeout = "20s"
production = false
debug = false
address = ":8000"
crt = "./tmp/kolide.crt"
key = "./tmp/kolide.key"
  ```

## osqueryd

  ```bash
  #!/usr/bin/env bash

  SECRET="kolidedev"
  SERVER="localhost:8000"
  CERT="./tmp/kolide.crt"

  echo $PWD

  sudo -E osqueryd \
    --verbose \
    --pidfile /tmp/osquery.pid \
    --host_identifier uuid \
    --database_path /tmp/osquery.db \
    --config_plugin tls \
    --config_tls_endpoint /api/v1/osquery/config \
    --config_tls_refresh 10 \
    --config_tls_max_attempts 3 \
    --enroll_tls_endpoint /api/v1/osquery/enroll  \
    --enroll_secret_env SECRET \
    --disable_distributed=false \
    --distributed_plugin tls \
    --distributed_interval 10 \
    --distributed_tls_max_attempts 3 \
    --distributed_tls_read_endpoint /api/v1/osquery/read \
    --distributed_tls_write_endpoint /api/v1/osquery/write \
    --tls_dump true \
    --logger_path /tmp/ \
    --logger_plugin tls \
    --logger_tls_endpoint /api/v1/osquery/log \
    --logger_tls_period 5 \
    --tls_hostname $SERVER \
    --tls_server_certs $CERT \
    --pack_delimiter /
  ```

# Contributing
When contributing to this repository, please first discuss the change you wish to make via issue,
email, or any other method with the owners of this repository before making a change.

## Pull Request Process

1. Ensure any install or build dependencies are removed before the end of the layer when doing a
   build.
2. Update the README.md with details of changes to the interface, this includes new environment
   variables, exposed ports, useful file locations and container parameters.
3. Increase the version numbers in any examples files and the README.md to the new version that this
   Pull Request would represent. The versioning scheme we use is [SemVer](http://semver.org/).
4. You may merge the Pull Request in once you have the sign-off of two other developers, or if you
   do not have permission to do that, you may request the second reviewer to merge it for you.

# Self-Promotion
Like kolide? Follow us on:
[GitHub](https://github.com/kolide)
[Twitter](http://twitter.com/kolideco)

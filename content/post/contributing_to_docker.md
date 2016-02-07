+++
categories = ["misc"]
date = "2016-02-07T01:15:35+01:00"
title = "Contributing to Docker"
+++

This is a short list of steps on how to contribute to the [Docker](https://github.com/docker/docker) project.

First you have to claim an issue and clone or update the repo.

- find an issue and claim it with `#dibs`
- fork it on [Github](https://github.com/docker/docker)
- clone the repo
	- `git clone https://github.com/<your name>/docker`
- change directory to the docker fork and `git checkout master`
- if branch `upstream` is missing, add it
	- `git remote add upstream https://github.com/docker/docker.git`
- update the repo:
	- `git fetch upstream master`
	- `git rebase upstream/master`
- create a new feature branch (`XXXX` stands for the issue number):
	- `git checkout -b XXXX-description`
- rebase your branch from master:
	- `git rebase upstream/master`

You are now up-to-date and on your feature branch. You can start working on the issue and make some changes. You should open three
terminals and start a VM with `docker-machine start <name>` if you are on OS X.

- terminal 1:
	- `eval "$(docker-machine env <name>)" # if you are on OS X`
	- `docker build -t XXXX-description . # creates a Docker dev env image`
	- `docker run --privileged --rm -ti -v $(pwd):/go/src/github.com/docker/docker XXXX-description /bin/bash`
	- Now you run inside your dev env container.
	- `hack/make.sh binary # builds Docker inside a container`
	- `cp bundles/latest/binary/docker /usr/bin`
	- `docker daemon -D # runs Docker inside a Docker container in debug mode`
- terminal 2:
	- `eval "$(docker-machine env <name>)" # if you are on OS X`
	- `docker ps # look for the name of your previously built container`
	- `docker exec -it <name-of-container> /bin/bash`
	- Now you have two terminals open in your dev container.
	- `docker run hello-world # run some container on your dev daemon`
- terminal 3:
	- make some changes to the code
	- in terminal 1: rebuild Docker: `hack/make.sh binary`
	- in terminal 1: copy the binary: `cp bundles/latest/binary/docker /usr/bin`
	- in terminal 1: restart the daemon with `docker daemon -D`
	- in terminal 2: run another container

Now you should run the tests to see whether they still pass.

- `make test` or `make test-unit`
- you may run the integration tests in the directory integration-cli:
	- `TESTFLAGS='-check.f TestXXX' make BINDIR=. test-integration-cli`
	- `TESTFLAGS='-check.f TestXXX'` allows you to run tests selectively

If all tests pass, you can commit and push your changes and create a pull request.

- `git commit -s -m 'Fixes #XXXX. ...'`
- `git push origin XXXX-description`
- create a pull request from your feature branch `XXXX-description`

If you make further changes, don't forget to update your branch by rebasing it from master.

- change some code
- `git commit -s -m 'Fixes #XXXX. ...'`
- `git fetch upstream master`
- `git rebase upstream/master` or `git rebase -i HEAD~<n>`
- `git push -f origin XXXX-description # update the branch of your pull request`

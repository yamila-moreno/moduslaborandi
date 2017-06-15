---
author: yamila
comments: true
date: 2017-06-15 10:02:22+00:00
slug: gitlabci-101-ssh-keys
title: Gitlab CI 101 - SSH keys
tags:
- docker
- gitlab-ci
- gitlab
- continuous-integration
---

After learning <a href="" target="_new">the basis of Gitlab CI</a>, I found a very specific need: use the gitlabci jobs to automatically tag the repository. To accomplish this, I had to configure several things related to ssh keys, and I guess that maybe you will find it useful.
<!--more-->

<h2>Assumptions</h2>

At least you followed the tutorial about <a href="" target="_new">the basis of Gitlab CI</a> and you know something about ssh key pairs.

<h2>Detailed goal</h2>

At the end of the post you will be able to create a tag in a gitlabci job and push it to the main repository. This is a little change from where we finish the introduction, and although is simple, it's also labourious, and it's convenient to have a cheat sheet close at hand.

<h2>Configure Gitlab CI</h2>

First of all, you need a pair of ssh keys.

{{< alert danger >}} Do not use  your existing keys; create new keys only for Gitlab, so you can revoke them easily if needed. {{< /alert >}}

```
$ ssh-keygen -t rsa -C "agent@gitlab.com" -b 4096
```

Now we have to upload both keys in Gitlab CI:

We have to add the **publick key** in <em>Project > Settings > Repository > Deploy Keys</em> and we give it writing permissions.

<figure>
<img src="https://c1.staticflickr.com/5/4258/35281118396_85ca264914_b.jpg" alt="Screenshot of gitlab ci deploy keys setting">
<figcaption>Upload here the public key</figcaption>
</figure>


Likewise, we have to add the **private key** in <em>Project > Settings > CI_CD Pipelines > Secret Variables</em>, where we add a meaningful name, as <code>GIT_SSH_KEY</code>. This secret is accesible from <code>.gitlab-ci.yml</code> as an environment variable.

<figure>
<img src="https://c1.staticflickr.com/5/4204/34511058103_316b8f8d21_b.jpg" alt="Screenshot of gitlab ci secrets setting">
<figcaption>Upload here the private key</figcaption>
</figure>

From now, Gitlab agents will load the private key, which comes from the secret, and they wiill have ssh access to the Gitlab "host", where we loaded a deploy key.

<h2>Configure .gitlab-ci.yml</h2>

Once we have GitlabCi properly configured, we'll add a new job that pushes the tag. My <code>.gitlab-ci.yml</code> is something like this:

```
image: docker

services:
- docker:dind

variables:
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

stages:
  - test
  - tag

before_script:
  - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY

create-library:
  stage: test
  script:
    - docker build -t $IMAGE_TAG .
    - docker run $IMAGE_TAG pytest
    - docker push $IMAGE_TAG
  only:
    - master

# This job is only executed if stage test finishes ok
tag-version:
  stage: tag
  script:

    # install the command line tool ssh-client, to manage private keys
    - apk update && apk add git openssh-client

    # activate the ssh-agent
    - eval $(ssh-agent -s)

    # load the private key, which is accesible as a environment variable
    - echo "$GIT_SSH_KEY" | ssh-add -
    - mkdir -p ~/.ssh

    # ensure that ssh will trust a new host, instead of asking
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

    # we also need to configure name and email for git user
    - git config user.name "Gitlab Agent"
    - git config user.email "agent@gitlab.com"

    # the repo is initially cloned with https so we change the
    # remote origin to point to the ssh access
    - git remote set-url origin git@gitlab.com:yamila/calculus.git

    # finally, we can use git normally. In this case, we are adding a tag with a comment
    - git tag -a $IMAGE_TAG -m "Hello, there"

    # and publishing the tags
    - git push origin --tags

  # we are executing this job only for the items matching this regular expression
  # in this case, the regular expression means "everything"
  only:
    - /^*$/

  # besides, we do not want to create a tag every time a tag is pushed, so we exclude
  # this job when a new tag is created
  except:
    - tags
```

And that's all! You can see that the logic behind is quite simple, but the commands are a bit tedious. Now, I'm using the ssh keys to push new tags to the repository, but another scenario could be use them to deploy the new version in our development server for instance.

Happy hacking!

# Download Private Repository in Docker Image

Let's see how a private repository can be downloaded inside Docker image and build code from there.

## `git clone` over SSH

This tutorial will focus on downloading code repository in Docker container by `git clone` command over [SSH protocol](https://www.ssh.com/academy/ssh/protocol). This is handy because this process is not only secure, but also it won't prompt for username and password. For the purpose of this article, the private repository [codingame](https://bitbucket.org/donotalo/codingame/src/master/) will be used. Naturally, because it's a private repository, it can't be accessed by the audience of this tutorial. It is suggested to use a private repository that the reader has access to complete this tutorial.

### Pre-requisite

To download git repository over SSH, SSH keys need to be generated, they need to be added by SSH agent and the public key needs to be added in git remote repository. How these can be achieved are Operating System and git repository host dependent. The audience are advised to learn on their own.

### Dockerfile

Change the `git clone` statement in the docker file with the following:

```
RUN --mount=type=ssh git clone git@bitbucket.org:donotalo/codingame.git
```
- 

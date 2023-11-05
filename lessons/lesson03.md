# Download Private Repository in Docker Image

Let's see how a private repository can be downloaded inside Docker image and build code from there.

## `git clone` over SSH

A private repository can be downloaded in local machine by `git clone` command over [SSH protocol](https://www.ssh.com/academy/ssh/protocol). This is handy because it won't ask for username and password. For the purpose of this article, the private repository [codingame](https://bitbucket.org/donotalo/codingame/src/master/) will be used. Naturally, because it's a private repository, it can't be accessed by the audience of this tutorial. It is suggested that you use your own private repository for the sake of this article.

### Pre-requisite

To download over SSH, SSH keys need to be generated and they need to be added by SSH agent.

```
RUN -mount=type-ssh git clone git@bitbucket.org:donotalo/codingame.git
```

# docker-sample

Some notes and samples about Dockerfile and docker-compose:

### dockerfile-entrypoint-cmd: How to pass CMD argument & Dockerfile ENV to ENTRYPOINT bash shell

When we write Dockerfile, sometime we need to inject ENV and CMD to the command run in docker container. But because we cannot pass direct `ENV` to `ENTRYPOINT` like this:

```dockerfile
ENV input_file=file1.txt

ENTRYPOINT ['tail', '$input_file']
CMD ['-n10']
```

Because, according docker committer: `They're not escaped perse, but the shell itself is what expands $ENV_VARS, so if you're bypassing the shell, it doesn't really make sense for environment variables to be expanded.` [Links](https://github.com/moby/moby/issues/4783#issuecomment-38244115)

if you use shell form of ENTRYPOINT [ENTRYPOINT has two forms](https://docs.docker.com/engine/reference/builder/#entrypoint), you can inject ENV to ENTRYPOINT

```dockerfile
ENV input_file=file1.txt

ENTRYPOINT tail $input_file
CMD ['-n10']
```

But if you use ENTRYPOINT as shell form, you cannot pass any parameter like CMD to behind ENTRYPOINT COMMAND: `The shell form prevents any CMD or run command line arguments from being used,` [Links](https://docs.docker.com/engine/reference/builder/#entrypoint).


My solution in case you need pass `ENV` value to command run in `ENTRYPOINT`, and you also need pass additional argument to docker conatainer run command like `CMD`, is put command in `ENTRYPOINT` to a bash script file `print_file.sh`:

```bash
#!/usr/bin/env bash

tail /root/$READ_FILE "$@"
```

and replace old command in `ENTRYPOINT` by bash command execute this script file:

```dockerfile
ENTRYPOINT ["/bin/sh", "/root/print_file.sh"]
CMD ["-v", "-n5"]
```

with this solution, in bash script file, you can inject ENV `$READFILE` create in Dockerfile to comand `tail`. Moreover, arguments we passed in CMD or in docker run will be used as parameter when exec `tail` command by use `$@` in tail command

To verify this solution:

- Build images from dockerfile:

```bash
cd dockerfile-entrypoint-cmd
docker build .  -t ubuntu/test-print-file:latest
```

- Run command with default `CMD`:

```bash
$ docker run ubuntu/test-print-file:latest                      
#Result:
==> /root/test.txt <==
6
7
8
9
10 
```

- Or run command with arguments passed to `docker run`:

```bash
$ docker run ubuntu/test-print-file:latest -n2                      
#Result:
9
10
```

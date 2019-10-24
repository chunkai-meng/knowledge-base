

- On Aliyun ECS

```
$ sudo apt update
$ sudo apt install snapd
# snap install docker
# scp ~/.ssh/id_rsa_git ali:~/.ssh
# chmod 600 id_rsa_git
```



- add github privit key

```shell
cat > config <<'EOF'
Host github.com
  HostName github.com
  IdentityFile ~/.ssh/id_rsa_git
EOF
```



- clone repo

  ```
  git clone git@github.com:chunkai-meng/dwhnt.git
  ```



- cd dwhnt

  ```
  docker-compose up -d dwhnt
  ```

  
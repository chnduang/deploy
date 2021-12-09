# oh-my-zsh
```
//warning like this；in ubuntu16.04；oh-my-zsh

zsh compinit: insecure directories, run compaudit for list.
Ignore insecure directories and continue [y] or abort compinit [n]? 

```

```
//There's a one line solution to fix that:一行命令解决这个问题

compaudit | xargs chmod g-w

```


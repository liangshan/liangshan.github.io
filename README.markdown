# liangshan.me

Source code of http://liangshan.me.

## Environments & Dependences

```
$ git clone git@github.com:liangshan/liangshan.github.io.git
$ cd liangshan.github.io
$ git checkout -B source origin/source

$ gem install bundler
$ rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
$ bundle install
```

## Contribute
```
$ rake new_post["title"]
$ rake genarate
$ rake preview
$ rake deploy
```


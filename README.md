# kubernetes-workshop

* <https://mzcdev.github.io/kubernetes-workshop/>

## setup

```bash
brew install hugo
```

## build

```bash
git submodule init ; git submodule update

npm install

rm -rf public && hugo -v
rm -rf docs && hugo -v -b /kubernetes-workshop/ -d docs
```

## localhost

```bash
hugo server -w -v --enableGitInfo --bind=0.0.0.0 --port 8080 --navigateToChanged
```

* <http://localhost:8080/>

# datascience

> src 폴더의 components와 page 폴더 내 코드를 보시면 될 것 같습니다.

## Commands

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# publish for production with express
npm run publish

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test

# deploy for github
npm run github-deploy
```


## Build for Dev
host & port setting

> /config/index.js
```
dev : {
  host : [HOST]
  port : [PORT]
}
```

## Build for Publish
port * cors setting

> /config/index.js
```
publish :{
  port : [PORT],
  cors : '*',
}
```


For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

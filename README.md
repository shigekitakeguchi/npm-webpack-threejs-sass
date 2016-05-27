# npmとwebpackで作るいい感じのthree.js開発環境 2016年5月の場合。

[npmとwebpackで作るいい感じのフロントエンド開発環境（React.js使わない）2016年5月の場合。](https://github.com/shigekitakeguchi/npm-webpack-sass)  
これをベースにしてthree.jsの開発環境。  
以前はGruntやGulpやらで用意してたけどnpm+webpackへ移行。

[three.js - Javascript 3D library](http://threejs.org/)  
WebGLをあつかう時にWebGLままで記述するのはちょっとツライので何かしらのフレームワークを利用すると思うけど（Unityでも作れるらしい）その中でもthree.jsが今のところメジャー。なので情報も多いし事例も多い。  
開発するってことになるとCDNから圧縮されたthree.min.jsを読み込むのもいいんだけどwebpackでrequireしてみました。

## 必要なものあるかどうか確認

```
node -v
```
まずはお決まりのNode.js入ってるか確認。

```
npm -v
```
npm（Node.jsのパッケージマネージャー）も入ってるか確認。

```
webpack -v
```
webpackも入っているか確認。  
もし入ってなかったら

```
npm install -g webpack
```
-gオプションはGlobalオプションのこと。

## ファイル・フォルダ構成

```
git clone https://github.com/shigekitakeguchi/npm-webpack-threejs-sass.git
```
[https://github.com/shigekitakeguchi/npm-webpack-threejs-sass](https://github.com/shigekitakeguchi/npm-webpack-threejs-sass)

GitHubから落として使ってください。  
カスタマイズなりなんなりして。

```
cd npm-webpack-sass
```
落としたフォルダ内に移動する。

```
├── README.md
├── app
│   ├── index.html
│   ├── scripts
│   └── styles
├── bs-config.json
├── package.json
├── src
│   ├── scripts
│   │   └── app.js
│   └── scss
│       ├── _normalize.css
│       └── style.scss
└── webpack.config.js
```

ファイル・フォルダ構成はこんな感じ。README.mdは不要。  
/src/scss/にnormalize.cssを含めた。resetがお好きならresetを。このあたりは好みで。  
/app/index.htmlを含めてる。内容は適宜変更ください。

```
npm install
```
これで必要なパッケージがインストールされるはず。

## package.json

```json
{
  "name": "npm-webpack-threejs-sass",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "dependencies": {
    "three": "^0.76.1"
  },
  "devDependencies": {
    "concurrently": "^2.1.0",
    "lite-server": "^2.2.0",
    "node-sass": "^3.7.0",
    "nodemon": "^1.9.2",
    "webpack": "^1.13.0"
  },
  "scripts": {
    "webpack": "webpack -w",
    "lite": "lite-server",
    "build-css": "node-sass ./src/scss/style.scss ./app/styles/style.css --output-style compressed",
    "watch-css": "nodemon -e scss -x \"npm run build-css\"",
    "start": "concurrently \"npm run lite\" \"npm run webpack\" \"npm run watch-css\""
  },
  "keywords": [],
  "author": "shigeki.takeguchi",
  "license": "MIT"
}
```
中身はこんな感じ。

### パッケージの説明

落としてきたpackage.jsonからインストールすればいいんですがそれぞれのパッケージの説明を。

```
npm install --save three
```
[https://github.com/mrdoob/three.js](https://github.com/mrdoob/three.js)

言わずもがななthree.js。requireしたいのでnpmしましょう。

```
npm install --save-dev concurrently
```
[https://github.com/kimmobrunfeldt/concurrently](https://github.com/kimmobrunfeldt/concurrently)

concurrentlyは複数のコマンド実行できるようにするため。具体的に何をしているかは後ほど説明。

```
npm install --save-dev lite-server
```
[https://github.com/johnpapa/lite-server](https://github.com/johnpapa/lite-server)

webpackにもwebpack dev serverというのがあるみたいだけどlite-serverのがシンプルで良さそうなので使ってみた。  
ただし設定ファイルは必要でした。

```
npm install --save-dev node-sass
```
[https://github.com/sass/node-sass](https://github.com/sass/node-sass)

node-sassでSassのコンパイル。全体的にNode.jsなのでSassのコンパイルだけRubyでなくてもいいだろうということで導入。

```
npm install --save-dev nodemon
```
[https://github.com/remy/nodemon](https://github.com/remy/nodemon)

scssを監視してコマンドを実行する。具体的に何をしているかは後ほど説明。

```
npm install --save-dev webpack
```
[https://github.com/webpack/webpack](https://github.com/webpack/webpack)

webpack。もうすでに何をするツールなのか説明しがたいくらい機能がある。  
静的なファイル（JavaScript系、CSS系、画像ファイル）の依存関係を解決するためのビルドツールってことなんだけど、ここでははJavaScriptだけを扱うようにしている。

## package.jsonの中のscriptsで何をしているか

```json
"scripts": {
  "webpack": "webpack -w",
  "lite": "lite-server",
  "build-css": "node-sass ./src/scss/app.scss ./app/styles/app.css --output-style compressed",
  "watch-css": "nodemon -e scss -x \"npm run build-css\"",
  "start": "concurrently \"npm run lite\" \"npm run webpack\" \"npm run watch-css\""
},
```

```
npm start
```
このコマンドでlite-serverを立ち上げwebpackでwatchを行いcssの変更を監視するようにしている。  

```json
"start": "concurrently \"npm run lite\" \"npm run webpack\" \"npm run watch-css\""
```
scriptsの中にあるstartがこれにあたる。

```
npm run lite
```
```
npm run webpack
```
```
npm run watch-css
```
これらのコマンドはそれぞれ独立したコマンドですが、最初にちょっと触れたがconcurrentlyにダブルクオーテーションでくくってスペースで区切って引数で渡せば並行して実行することになる。便利。

```json
"webpack": "webpack -w",
```
これはwebpackのwatch（監視）を走らせている。こちらも後ほど触れるがwebpack.config.jsonで記述されたことをもとに監視している。

```json
"lite": "lite-server",
```
lite-serverを立ち上げている。bs-config.jsonに設定ないようを記述している（こちらも後ほど触れる）。

```json
"build-css": "node-sass ./src/scss/app.scss ./app/styles/app.css --output-style compressed",
```
Sass（Scss）をnode-sassを使ってコンパイルしている。  

* ./src/scss/app.scssはコンパイルする前のファイル。css（個人的にはScss記法だけど）の記述はこれにする。
* ./app/styles/app.cssはコンパイル後のファイル。
* --output-style compressedはコンパイル後のファイルを圧縮する設定。他にはnested、expanded（これが一般的に人が書くのに近いスタイル）、compactが使える。これはガイドラインや好みで。

[Sass Documentation(output_style)](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#output_style)

```json
"watch-css": "nodemon -e scss -x \"npm run build-css\"",
```
nodemonを使ってscssファイルを監視し、変更があれば「npm run build-css」を走らせるという設定。  
「-e scss」ってのがscssを監視するというオプション。-xは「npm run build-css」を実行するためのオプションになる。

## webpack.config.json
```javascript
var webpack = require("webpack");

module.exports = {
  entry: './src/scripts/app.js',
  output: {
    path: __dirname + '/app/js',
    filename: 'bundle.js',
    publicPath: '/app/',
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin()
  ]
}
```
webpackの設定はいたってシンプル。entryにもとファイル（複数ある場合は配列で持たせる）。outputに出力される設定を記述。今回はbundle.jsっていう一般的によく使われているらしい名称のまま。  
素のままのファイルだともうファイルがでかくてあれなんでUglifyJsPluginで圧縮・最適化。

## bs-config.json

```json
{
  "injectChanges": "true",
  "files": ["./app/**/*.{html,htm,css,js}"],
  "watchOptions": { "ignored": "node_modules" },
  "server": { "baseDir": "./app" }
}
```
lite-serverの設定はドキュメントルートをappの直下にしたかったのと監視対象のファイル（html、css、js）が変更されたらリロードしてinjectChangesというBrowsersyncを動すため。

## three.jsを使っての記述

```javascript
var THREE = require('three');

(function(){
  var scene, camera, renderer;
  var geometry, material, mesh;

  init();
  animate();

  function init() {

      scene = new THREE.Scene();

      camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 1, 10000 );
      camera.position.z = 1000;

      geometry = new THREE.BoxGeometry( 200, 200, 200 );
      material = new THREE.MeshBasicMaterial( { color: 0xff0000, wireframe: true } );

      mesh = new THREE.Mesh( geometry, material );
      scene.add( mesh );

      renderer = new THREE.WebGLRenderer();
      renderer.setSize( window.innerWidth, window.innerHeight );

      document.body.appendChild( renderer.domElement );

  }

  function animate() {

      requestAnimationFrame( animate );

      mesh.rotation.x += 0.01;
      mesh.rotation.y += 0.02;

      renderer.render( scene, camera );

  }
})();
```
[https://github.com/mrdoob/three.js](https://github.com/mrdoob/three.js)

このページのサンプルを記述するとしたらこんな感じになる。    
即時関数でくくるのは好みの問題。なくてももちろん動きます。

```javascript
var THREE = require('three');
```
こうやってrequireすることの他はとくに設定は不要。

---

まだwebpackをはじめて数日とかという状態なので間違えや指摘をいただけると助かります。
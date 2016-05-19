# Testowanie aplikacji stworzonych w technologii React Native
*Autor: Grzegorz Świrski*<br />
*19 maja 2016*

## Czym jest React Native
React Native to technologia stworzona przez firmę Facebook, Inc. do tworzenia aplikacji mobilnych.
Umożliwia ona pisanie przenośnych programów na platformę iOS i Android przy użyciu narzędzi znanych


## Zakres projektu

## Instalacja
0. Zainstaluj React Native podążając za instrukcjami: https://facebook.github.io/react-native/docs/getting-started.html
1. Stwórz nowy projekt w React Native:

    ```
    react-native init ExampleProject && mkdir ExampleProject/src && mkdir -p ExampleProject/test/setup
    ```
2. Zainstaluj niezbędne zależności:
  - `npm install --save-dev mocha` - biblioteka do organizacji testów
  - `npm install --save-dev chai` - biblioteka do definiowania wymaganych rezultatów w testach
  - `npm install --save-dev enzyme` - pomocnicza biblioteka do testowania aplikacji stworzonych w React
  - `npm install --save-dev react-dom` - pomocnicza biblioteka do testowania aplikacji stworzonych w React
  - `npm install --save-dev gswirski/react-native-mock` - zestaw "mocków" dla React Native.
3. Stwórz plik `test/setup/compiler.js`:

    ```js
    var fs = require('fs');
    var origJs = require.extensions['.js'];
    var babel = require('babel-core');
    
    require.extensions['.png'] = function (module, fileName) {
      return null;
    };
    
    require.extensions['.js'] = function (module, fileName) {
      if (fileName.indexOf('node_modules/') >= 0) {
        return (origJs || require.extensions['.js'])(module, fileName);
      }
      var src = fs.readFileSync(fileName, 'utf8');
      output = babel.transform(src, {
        filename: fileName,
        sourceFileName: fileName,
    
        //keep below in sync with babelrc
        retainLines: true,
        compact: true,
        comments: false,
        presets: ["react-native"],
        plugins: [],
        sourceMaps: false
      }).code;
    
      return module._compile(output, fileName);
    };
    ```
4. Uzupełnij sekcję "scripts" w `package.json`:

    ```
    "scripts": {
      "start": "node node_modules/react-native/local-cli/cli.js start",
      "test": "mocha --require react-native-mock/mock.js --compilers js:test/setup/compiler.js --recursive test/*.js"
    },
    ````
    
## Implementacja projektu

### Wsparcie dla React Native 0.26.0
0.26.0 jest aktualną wersją w momencie pisania tej pracy.

```diff
From 9d902226e565725daf2b35b1627e3471a2ba98b3 Mon Sep 17 00:00:00 2001
From: =?UTF-8?q?Grzegorz=20=C5=9Awirski?= <grzegorz@swirski.name>
Date: Thu, 19 May 2016 18:01:20 +0200
Subject: [PATCH] update for RN 0.26.0

---
 .babelrc            | 27 ++-------------------------
 package.json        | 19 +++++++++----------
 src/react-native.js |  1 -
 3 files changed, 11 insertions(+), 36 deletions(-)

diff --git a/.babelrc b/.babelrc
index 36f41c0..3c3192a 100644
--- a/.babelrc
+++ b/.babelrc
@@ -1,27 +1,4 @@
 {
-  "presets": ["airbnb"],
-  "plugins": [
-    "syntax-async-functions",
-    "syntax-class-properties",
-    "syntax-trailing-function-commas",
-    "transform-class-properties",
-    "transform-es2015-arrow-functions",
-    "transform-es2015-block-scoping",
-    "transform-es2015-classes",
-    "transform-es2015-computed-properties",
-    "transform-es2015-constants",
-    "transform-es2015-destructuring",
-    ["transform-es2015-modules-commonjs", { "strict": false, "allowTopLevelThis": true }],
-    "transform-es2015-parameters",
-    "transform-es2015-shorthand-properties",
-    "transform-es2015-spread",
-    "transform-es2015-template-literals",
-    "transform-flow-strip-types",
-    "transform-object-assign",
-    "transform-object-rest-spread",
-    "transform-react-display-name",
-    "transform-react-jsx",
-    "transform-regenerator",
-    "transform-es2015-for-of"
-  ]
+  "presets": ["react-native"],
+  "plugins": []
 }
diff --git a/package.json b/package.json
index 3e21b06..05d3134 100644
--- a/package.json
+++ b/package.json
@@ -51,27 +51,26 @@
     "babel-plugin-transform-react-display-name": "^6.4.0",
     "babel-plugin-transform-react-jsx": "^6.4.0",
     "babel-plugin-transform-regenerator": "^6.4.4",
-    "babel-preset-airbnb": "^1.0.1",
+    "babel-preset-react-native": "^1.7.0",
     "chai": "^3.5.0",
     "eslint": "^1.10.3",
     "eslint-config-airbnb": "^4.0.0",
     "eslint-plugin-react": "^3.16.1",
     "mocha": "^2.4.5",
-    "react": "^0.14.7",
-    "react-native": "^0.18.1"
+    "react": "^15.0.2",
+    "react-native": "^0.26.0"
   },
   "dependencies": {
     "cubic-bezier": "^0.1.2",
     "invariant": "^2.2.0",
     "keymirror": "^0.1.1",
     "raf": "^3.1.0",
-    "react-addons-clone-with-props": "^0.14.7",
-    "react-addons-create-fragment": "^0.14.7",
-    "react-addons-linked-state-mixin": "^0.14.7",
-    "react-addons-perf": "^0.14.7",
-    "react-addons-pure-render-mixin": "^0.14.7",
-    "react-addons-test-utils": "^0.14.7",
-    "react-addons-update": "^0.14.7",
+    "react-addons-create-fragment": "^15.0.2",
+    "react-addons-linked-state-mixin": "^15.0.2",
+    "react-addons-perf": "^15.0.2",
+    "react-addons-pure-render-mixin": "^15.0.2",
+    "react-addons-test-utils": "^15.0.2",
+    "react-addons-update": "^15.0.2",
     "react-timer-mixin": "^0.13.3",
     "warning": "^2.1.0"
   },
diff --git a/src/react-native.js b/src/react-native.js
index e49d36e..b8b1d69 100644
--- a/src/react-native.js
+++ b/src/react-native.js
@@ -107,7 +107,6 @@ const ReactNativeAddons = {
   TestUtils: require('react-addons-test-utils'),
   // TODO(lmr): not sure where to find this
   // batchedUpdates: require('ReactUpdates').batchedUpdates,
-  cloneWithProps: require('react-addons-clone-with-props'),
   createFragment: require('react-addons-create-fragment'),
   update: require('react-addons-update'),
 };
-- 
2.7.4 (Apple Git-66)
```

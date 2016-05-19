# Testowanie aplikacji stworzonych w technologii React Native
*Autor: Grzegorz Świrski*<br />
*19 maja 2016*

## Czym jest React Native
React Native to technologia stworzona przez firmę Facebook, Inc. do tworzenia aplikacji mobilnych.
Umożliwia ona pisanie przenośnych programów na platformę iOS i Android przy użyciu narzędzi znanych


## Zakres projektu

## Instalacja
0. Zainstaluj React Native podążając za instrukcjami: https://facebook.github.io/react-native/docs/getting-started.html
1. Stwórz nowy projekt w React Native: `react-native init ExampleProject && mkdir ExampleProject/src && mkdir -p ExampleProject/test/setup`
2. Zainstaluj niezbędne zależności:
  - `npm install --save-dev mocha` - biblioteka do organizacji testów
  - `npm install --save-dev chai` - biblioteka do definiowania wymaganych rezultatów w testach
  - `npm install --save-dev enzyme` - pomocnicza biblioteka do testowania aplikacji stworzonych w React
  - `npm install --save-dev react-dom` - pomocnicza biblioteka do testowania aplikacji stworzonych w React
  - `npm install --save-dev gswirski/react-native-mock` - zestaw "mocków" dla React Native.
3. Stwórz plik `test/setup/compiler.js`:
    ```js
    var fs = require('fs');
var path = require('path');
var origJs = require.extensions['.js'];
var babel = require('babel-core');
var transformer = require('react-native/packager/transformer.js');

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
    "retainLines": true,
    "compact": true,
    "comments": false,
    "presets": ["react-native"],
    "plugins": [],
    "sourceMaps": false
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

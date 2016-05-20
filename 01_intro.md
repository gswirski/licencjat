***Uniwersytet Jagielloński***<br />
*Wydział Matematyki i Informatyki*<br />
*Zespół Katedr i Zakładów Informatyki Matematycznej*

### Grzegorz Świrski
# React Native - automatyczne testy funkcjonalne
**Praca licencjacja na kierunku INFORMATYKA ANALITYCZNA**

*Opiekun pracy: dr Grzegrz Gutowski*

*Kraków, 2016*


## O projekcie React Native
React Native to technologia służąca do tworzenia przenośnych aplikacji mobilnych. Programy napisane przy
użyciu React Native działają w środowisku JavaScript co pozwala wykorzystywać spory zasób bibliotek do
automatycznego testowania i narzędzi do debugowania stworzonych dla tego języka. Wynikowy program
wykorzystuje komponenty UI systemu operacyjnego na którym się wykonuje tym samym dostarczając użytkownikowi
natywną jakość interfejsu.

## O testach funkcjonalnych
Testy automatyczne weryfikują, czy program działa poprawnie. Dzięki temu po wprowadzeniu zmian w aplikacji
jesteśmy w stanie szybko sprawdzić, czy nie zepsuliśmy jakiejś innej, wcześniej działającej logiki.
Testy funkcjonalne działają na poziomie end-to-end. Zaczynają od pewnego widoku użytkownika, symulują
akcje użytkownika (np. kliknięcie w element tekstowy) po czym sprawdzają, czy wynikowy widok jest poprawny.
Przykładowo, czy po naciśnięciu przycisku "dodaj komentarz" komentarz faktycznie wyświetla się
na stronie.

## Zakres projektu
Istnieje bardzo dużo narzędzi do automatycznego testowania kodu stworzonych dla języka JavaScript.
Niestety, ponieważ React Native musi skomunikować się z systemem operacyjnym iOS lub Android, to uruchamianie
tych testów na komputerze jest bardzo problematyczne. Symulowanie mobilnego systemu operacyjnego (albo testowanie
na zewnętrznym urządzeniu) wprowadza spory narzut czasowy.

Na szczęście React Native oparty jest o inną, przeglądarkową wersję Reacta. Dzięki temu możemy próbować zastąpić
natywne elementy interfejsu (czyli te elementy, gdzie React Native oczekuje mobilnego systemu operacyjnego) przez
widoki HTML-owe (czyli te używane w przeglądarce). Takie podejście pozwala nam przybliżyć sposób działania programu
i przetestować czy wszystkie istotne informacje faktycznie zostały wyświetlone na ekranie i czy interakcje
z użytkownikiem zachowują się tak jak powinny.

`react-native-mock` to biblioteka, która implementuje takie właśnie podejście - zastępowanie komponentów natywnych
przez komponenty HTML-owe. Ta praca rozszerza działanie biblioteki o możliwość wyświetlania pełnej struktury
widoku i wchodzenia w interakcje z elementami (kliknięcia). Poprawione zostało również wsparcie dla aktualnej
wersji React Native.

## Przykład

Weźmy program, który dostaje tablicę `string`ów (wejście) i zwraca widok listy (natywny komponent)
z wyświetlonymi wszystkimi elementami tablicy. Dodatkowo będziemy chcieli, żeby domyślnie wszystkie litery
były wielkie, natomiast po kliknięciu na element wszystkie litery (tego konkretnego elementu) były małe.
Implementacja takiego komponentu w React Native znajduje się poniżej.

**src/ListComponent.js**
```js
import React, { Component } from 'react';
import { ListView, Text } from 'react-native';

class CapitalizedText extends Component {
    constructor(props) {
        super(props);
        this.state = { text: props.text.toUpperCase() };
    }
    
    onPress = () => {
        const text = this.state.text.toLowerCase();
        this.setState({ text }};
    };
    
    render() {
        return <Text onPress={this.onPress}>{this.state.text}</Text>;
    }
}

export default class ListComponent extends Component {
    constructor(props) {
        super(props);
        let ds = new ListView.DataSource({ rowHasChanged: (r1, r2) => r1 != r2 });
        this.state = {
            dataSource = ds.cloneWithRows(props.items),
        };
    }
    
    renderRow = (rowData) => {
        return <CapitalizedText text={rowData} />;
    };
    
    render() {
        return (
            <ListView
                dataSource={this.state.dataSource}
                renderRow={this.renderRow} />
        );
    }
}
```

Dzięki wprowadzonym zmianom w bibliotece `react-native-mock` przetestowanie takiego kodu staje się
możliwe. Poniższy test sprawdza, czy widok listy faktycznie zawiera wszystkie przekazane elementuy i każdy z nich
napisany jest wielkimi literami. Sprawdza również, czy po kliknięciu na któryś z elementów listy 
litery zostają zamienione na małe.

**test/ListComponent.js**

```js
import jsdom from 'mocha-jsdom';
import { mount } from 'enzyme';
import { expect } from 'chai';
import React from 'react';
import ListComponent from '../src/ListComponent';

describe('<ListComponent />', () => {
  jsdom();

  it ('renders list with capitalized labels', () => {
    const data = ['lorem', 'ipsum', 'dolor'];
    const wrapper = mount(<ListComponent items={data} />);
    const html = wrapper.html();

    expect(wrapper.find('[data-rn-name="ListViewRow"]').length).to.equal(3);
    expect(wrapper.html()).to.contain('LOREM');
    expect(wrapper.html()).to.contain('IPSUM');
    expect(wrapper.html()).to.contain('DOLOR');
  });

  it('responds to clicks', () => {
    const data = ['lorem', 'ipsum', 'dolor'];
    const wrapper = mount(<ListComponent items={data} />);
    wrapper.find('[data-rn-name="ListViewRow"] Text').at(1).simulate('click');
    const html = wrapper.html();
    expect(wrapper.html()).to.contain('ipsum');
  });
});

```

## Instalacja
1. Zainstaluj React Native

    ```
    https://facebook.github.io/react-native/docs/getting-started.html
    ```

2. Stwórz nowy projekt

    ```
    react-native init ExampleProject && mkdir ExampleProject/src && mkdir -p ExampleProject/test/setup
    ```

3. Zainstaluj niezbędne zależności

    ```
    npm install --save-dev mocha # test-runner
    npm install --save-dev chai # assertions/expectations
    npm install --save-dev enzyme # zbiór helperów do testowania Reacta
    npm install --save-dev react-dom # zbiór helperów do testowania Reacta w środowisku DOM
    npm install --save-dev mocha-jsdom # implementacja WHATWG DOM na potrzeby testów automatycznych
    npm install --save-dev gswirski/react-native-mock # zestaw "mocków" dla React Native.
    ```

4. Stwórz plik `test/setup/compiler.js` (wsparcie dla składniowych rozszerzeń Reacta; transpiler do JavaScript)

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

5. Uzupełnij sekcję "scripts" w `package.json`

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
From: Grzegorz Swirski <grzegorz@swirski.name>
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

### Wyświetlanie struktury testowanych komponentów

Poniższe zmiany pozwalają wykonać funkcję `render()` na testowanych komponentach. Wynikiem
wywołania funkcji jest drzewo DOM zawierające całą strukturę aktualnego widoku aplikacji.
Dzięki tym poprawkom można zbadać czy wszystkie niezbędne informacje faktycznie są wyświetlane
na ekranie i czy dane pogrupowane są we właściwy, logiczny sposób.

```diff
From 1a00ab286fe8fd759bd3baed3ad3e03abcac0289 Mon Sep 17 00:00:00 2001
From: Grzegorz Swirski <grzegorz@swirski.name>
Date: Thu, 19 May 2016 18:48:04 +0200
Subject: [PATCH] render components, show display names in data-rn-name attr

---
 package.json                               |  1 +
 src/components/ActivityIndicatorIOS.js     |  2 +-
 src/components/Image.js                    |  2 +-
 src/components/ListView.js                 |  2 +-
 src/components/Navigator.js                |  2 +-
 src/components/ScrollView.js               |  2 +-
 src/components/Text.js                     |  2 +-
 src/components/TouchableWithoutFeedback.js |  2 +-
 src/components/View.js                     |  2 +-
 src/components/WebView.js                  |  2 +-
 src/components/createMockComponent.js      |  2 +-
 test/MockComponent.js                      | 18 ++++++++++++++++++
 test/basic-test.js                         |  1 -
 13 files changed, 29 insertions(+), 11 deletions(-)
 create mode 100644 test/MockComponent.js

diff --git a/package.json b/package.json
index 05d3134..85edabb 100644
--- a/package.json
+++ b/package.json
@@ -58,6 +58,7 @@
     "eslint-plugin-react": "^3.16.1",
     "mocha": "^2.4.5",
     "react": "^15.0.2",
+    "react-dom": "^15.0.2",
     "react-native": "^0.26.0"
   },
   "dependencies": {
diff --git a/src/components/ActivityIndicatorIOS.js b/src/components/ActivityIndicatorIOS.js
index 721682f..6b7a868 100644
--- a/src/components/ActivityIndicatorIOS.js
+++ b/src/components/ActivityIndicatorIOS.js
@@ -38,7 +38,7 @@ const ActivityIndicatorIOS = React.createClass({
     onLayout: PropTypes.func,
   },
   render() {
-    return null;
+    return <div data-rn-name="ActivityIndicatorIOS" />;
   },
 });
 
diff --git a/src/components/Image.js b/src/components/Image.js
index 3b9b9d8..df21678 100644
--- a/src/components/Image.js
+++ b/src/components/Image.js
@@ -114,7 +114,7 @@ var Image = React.createClass({
     },
   },
   render() {
-    return null;
+    return <div data-rn-name="Image" />;
   },
 });
 
diff --git a/src/components/ListView.js b/src/components/ListView.js
index a4886ca..11924d7 100644
--- a/src/components/ListView.js
+++ b/src/components/ListView.js
@@ -178,7 +178,7 @@ const ListView = React.createClass({
   },
 
   render() {
-    return null;
+    return <div data-rn-name="ListView" />;
   },
 });
 
diff --git a/src/components/Navigator.js b/src/components/Navigator.js
index 14a1d57..5649f29 100644
--- a/src/components/Navigator.js
+++ b/src/components/Navigator.js
@@ -96,7 +96,7 @@ const Navigator = React.createClass({
     SceneConfigs: NavigatorSceneConfigs,
   },
   render() {
-    return null;
+    return <div data-rn-name="Navigator" />;
   }
 });
 
diff --git a/src/components/ScrollView.js b/src/components/ScrollView.js
index 15759da..f14cd74 100644
--- a/src/components/ScrollView.js
+++ b/src/components/ScrollView.js
@@ -312,7 +312,7 @@ const ScrollView = React.createClass({
   },
 
   render() {
-    return null;
+    return <div data-rn-name="ScrollView">{this.props.children}</div>;
   },
 });
 
diff --git a/src/components/Text.js b/src/components/Text.js
index a0c4fc4..e4d464c 100644
--- a/src/components/Text.js
+++ b/src/components/Text.js
@@ -45,7 +45,7 @@ const Text = React.createClass({
     allowFontScaling: React.PropTypes.bool,
   },
   render() {
-    return null;
+    return <div data-rn-name="Text">{this.props.children}</div>;
   },
 });
 
diff --git a/src/components/TouchableWithoutFeedback.js b/src/components/TouchableWithoutFeedback.js
index fcc21e3..26421d0 100644
--- a/src/components/TouchableWithoutFeedback.js
+++ b/src/components/TouchableWithoutFeedback.js
@@ -71,7 +71,7 @@ const TouchableWithoutFeedback = React.createClass({
     hitSlop: EdgeInsetsPropType,
   },
   render() {
-    return null;
+    return <div data-rn-name="TouchableWithoutFeedback">{this.props.children}</div>;
   },
 });
 
diff --git a/src/components/View.js b/src/components/View.js
index 1ca0fe0..493f4a7 100644
--- a/src/components/View.js
+++ b/src/components/View.js
@@ -278,7 +278,7 @@ const View = React.createClass({
     needsOffscreenAlphaCompositing: PropTypes.bool,
   },
   render() {
-    return null;
+    return <div data-rn-name="View">{this.props.children}</div>;
   },
 });
 
diff --git a/src/components/WebView.js b/src/components/WebView.js
index fdfe74a..366f1e7 100644
--- a/src/components/WebView.js
+++ b/src/components/WebView.js
@@ -137,7 +137,7 @@ const WebView = React.createClass({
     return React.findNodeHandle(this.refs[RCT_WEBVIEW_REF]);
   },
   render() {
-    return null;
+    return <div data-rn-name="WebView" />;
   },
 });
 
diff --git a/src/components/createMockComponent.js b/src/components/createMockComponent.js
index f7b1a32..de97fbe 100644
--- a/src/components/createMockComponent.js
+++ b/src/components/createMockComponent.js
@@ -4,7 +4,7 @@ function createMockComponent(displayName) {
   return React.createClass({
     displayName,
     render() {
-      return null;
+      return <div data-rn-name={displayName}>{this.props.children}</div>;
     },
   });
 }
diff --git a/test/MockComponent.js b/test/MockComponent.js
new file mode 100644
index 0000000..d31f7d9
--- /dev/null
+++ b/test/MockComponent.js
@@ -0,0 +1,18 @@
+import { expect } from 'chai';
+import createMockComponent from '../src/components/createMockComponent';
+import React, { Text } from '../src/react-native';
+import ReactDOMServer from 'react-dom/server';
+
+
+describe('createMockComponent', () => {
+  it ('renders component structure', () => {
+    const Container = createMockComponent('Container');
+
+    let component = <Container><Text>Hello World!</Text></Container>;
+    let html = ReactDOMServer.renderToString(component);
+
+    expect(html).to.contain('data-rn-name="Container"');
+    expect(html).to.contain('data-rn-name="Text"');
+    expect(html).to.contain('Hello World!');
+  });
+});
diff --git a/test/basic-test.js b/test/basic-test.js
index 4a27f79..0c80a25 100644
--- a/test/basic-test.js
+++ b/test/basic-test.js
@@ -3,7 +3,6 @@ import { expect } from 'chai';
 
 describe('Requires', () => {
   it('requires', () => {
-    console.log(Object.keys(React));
     expect(true).to.equal(true);
   });
 });
-- 
2.7.4 (Apple Git-66)
```

### Wsparcie dla `ListView`
Aby zapewnić wysoką wydajność, `ListView` korzysta z niestandardowego mechanizmu przekazywania
modelu do widoku listy. Ta poprawka jest kontynuacją pracy nad generowaniem pełnego drzewa DOM
aktualnie wyświetlanego widoku.

```diff
From 58084a6e18bf1f1939fd6f570bdf87ff4ee38f22 Mon Sep 17 00:00:00 2001
From: Grzegorz Swirski <grzegorz@swirski.name>
Date: Fri, 20 May 2016 11:53:18 +0200
Subject: [PATCH] render list mock

---
 src/api/ListViewDataSource.js | 60 ++++++++++++++++++++++++++++++------
 src/components/ListView.js    | 71 +++++++++++++++++++++++++++++++++++--------
 2 files changed, 108 insertions(+), 23 deletions(-)

diff --git a/src/api/ListViewDataSource.js b/src/api/ListViewDataSource.js
index 195f588..7804185 100644
--- a/src/api/ListViewDataSource.js
+++ b/src/api/ListViewDataSource.js
@@ -1,26 +1,66 @@
+function defaultGetRowData(dataBlob, sectionId, rowId) {
+  return dataBlob[sectionId][rowId];
+}
 
+function defaultGetSectionHeaderData(dataBlob, sectionId) {
+  return dataBlob[sectionId];
+}
 
 class ListViewDataSource {
-  constructor() {
+  constructor(params) {
+    this._config = {
+      rowHasChanged: params.rowHasChanged,
+      getRowData: params.getRowData || defaultGetRowData,
+      sectionHeaderHasChanged: params.sectionHeaderHasChanged,
+      getSectionHeaderData: params.getSectionHeaderData || defaultGetSectionHeaderData,
+    };
     this._dataBlob = null;
+    this._sectionIds = [];
+    this._rowIds = [];
   }
 
   getRowCount() {
 
   }
 
-  cloneWithRows(data) {
-    var newSource = new ListViewDataSource();
-    newSource._dataBlob = data;
+  cloneWithRows(dataBlob, rowIdentities) {
+    let rowIds = (rowIdentities) ? [rowIdentities] : null;
+
+    return this.cloneWithRowsAndSections({ s1: dataBlob }, ['s1'], rowIds);
+  }
+
+  cloneWithRowsAndSections(dataBlob, sectionIdentities, rowIdentities) {
+    var newSource = new ListViewDataSource(this._config);
+    newSource._dataBlob = dataBlob;
+
+    if (sectionIdentities) {
+      newSource._sectionIds = sectionIdentities;
+    } else {
+      newSource._sectionIds = Object.keys(dataBlob);
+    }
+
+    if (rowIdentities) {
+      newSource._rowIds = rowIdentities;
+    } else {
+      newSource._rowIds= [];
+      newSource._sectionIds.forEach((sectionId) => {
+        newSource._rowIds.push(Object.keys(dataBlob[sectionId]));
+      });
+    }
 
     return newSource;
   }
-  
-  cloneWithRowsAndSections(data) {
-      var newSource = new ListViewDataSource();
-      newSource._dataBlob = data;
-  
-      return newSource;
+
+  getSection(sectionId) {
+    return this._config.getSectionHeaderData(this._dataBlob, sectionId);
+  }
+
+  getRow(sectionId, rowId) {
+    return this._config.getRowData(this._dataBlob, sectionId, rowId);
+  }
+
+  getRowIds(sectionId) {
+    return this._rowIds[this._sectionIds.indexOf(sectionId)];
   }
 }
 
diff --git a/src/components/ListView.js b/src/components/ListView.js
index 11924d7..e3d2713 100644
--- a/src/components/ListView.js
+++ b/src/components/ListView.js
@@ -1,22 +1,11 @@
 import React from 'react';
+const { PropTypes } = React;
 import ScrollResponder from '../mixins/ScrollResponder';
 import TimerMixin from 'react-timer-mixin';
 import ScrollViewManager from '../NativeModules/ScrollViewManager';
 import ScrollView from './ScrollView';
 
 var ListViewDataSource = require('../api/ListViewDataSource');
-//var React = require('React');
-
-//var ScrollView = require('ScrollView');
-//var ScrollResponder = require('ScrollResponder');
-// var StaticRenderer = require('StaticRenderer');  // Unused
-//var TimerMixin = require('react-timer-mixin');
-
-// var isEmpty = require('isEmpty'); // Doesnt resolve
-// var logError = require('logError'); // Doesnt resolve
-//var merge = require('merge');
-
-const { PropTypes } = React;
 
 const DEFAULT_PAGE_SIZE = 1;
 const DEFAULT_INITIAL_ROWS = 10;
@@ -178,7 +167,63 @@ const ListView = React.createClass({
   },
 
   render() {
-    return <div data-rn-name="ListView" />;
+    let ds = this.props.dataSource;
+
+    let header = null;
+    if (this.props.renderHeader) {
+      header = <div data-rn-name="ListViewHeader">{this.props.renderHeader()}</div>;
+    }
+
+    let footer = null;
+    if (this.props.renderFooter) {
+      footer = <div data-rn-name="ListViewFooter">{this.props.renderFooter()}</div>;
+    }
+
+    let sections = ds._sectionIds.map((section) => {
+      let sectionData = ds.getSection(section);
+
+      let sectionHeader = null;
+      if (this.props.renderSectionHeader) {
+        sectionHeader = (
+          <div data-rn-name="ListViewSectionHeader">
+            {this.props.renderSectionHeader(sectionData, section)}
+          </div>
+        );
+      }
+
+      let rows = ds.getRowIds(section).map((row) => {
+        let rowData = ds.getRow(section, row);
+
+        let rowView = null;
+        if (this.props.renderRow) {
+          rowView = (
+            <div data-rn-name="ListViewRow" key={row}>
+              {this.props.renderRow(rowData, section, row)}
+            </div>
+          );
+        }
+
+        return rowView;
+      });
+
+      return (
+        <div data-rn-name="ListViewSection" key={section}>
+          {sectionHeader}
+          <div data-rn-name="ListViewSectionRows">
+            {rows}
+          </div>
+        </div>
+      );
+    });
+
+
+    return (
+      <div data-rn-name="ListView">
+        {header}
+        {sections}
+        {footer}
+      </div>
+    );
   },
 });
 
-- 
2.7.4 (Apple Git-66)
```

### Wsparcie dla `press events`
`press events` (dotknięcia ekranu przez użytkownika) są symulowane za pomocą `click events`
(użytkownik klika myszką). Dzięki takiemu podejściu możemy wykorzystywać biblioteki stworzone
do testowania HTML-owej wersji Reacta (m.in. enzyme użyte w przykładzie powyżej).

```diff
From e3f6f3b62a4513aaa1dcdbd7e588165833bbee0c Mon Sep 17 00:00:00 2001
From: Grzegorz Swirski <grzegorz@swirski.name>
Date: Fri, 20 May 2016 11:54:25 +0200
Subject: [PATCH] support press events in JSDOM env

---
 src/components/Text.js                     | 2 +-
 src/components/TouchableWithoutFeedback.js | 2 +-
 2 files changed, 2 insertions(+), 2 deletions(-)

diff --git a/src/components/Text.js b/src/components/Text.js
index e4d464c..457f2c4 100644
--- a/src/components/Text.js
+++ b/src/components/Text.js
@@ -45,7 +45,7 @@ const Text = React.createClass({
     allowFontScaling: React.PropTypes.bool,
   },
   render() {
-    return <div data-rn-name="Text">{this.props.children}</div>;
+    return <div data-rn-name="Text" onClick={this.props.onPress}>{this.props.children}</div>;
   },
 });
 
diff --git a/src/components/TouchableWithoutFeedback.js b/src/components/TouchableWithoutFeedback.js
index 26421d0..8f4883c 100644
--- a/src/components/TouchableWithoutFeedback.js
+++ b/src/components/TouchableWithoutFeedback.js
@@ -71,7 +71,7 @@ const TouchableWithoutFeedback = React.createClass({
     hitSlop: EdgeInsetsPropType,
   },
   render() {
-    return <div data-rn-name="TouchableWithoutFeedback">{this.props.children}</div>;
+    return <div data-rn-name="TouchableWithoutFeedback" onClick={onPress}>{this.props.children}</div>;
   },
 });
 
-- 
2.7.4 (Apple Git-66)
```

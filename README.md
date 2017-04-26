![XML ⇔ JS/JSON](http://nashwaan.github.io/xml-js/images/logo.svg)

Convert XML text to Javascript object / JSON text (and vice versa).

[![Build Status](https://ci.appveyor.com/api/projects/status/0ky9f115m0f0r0gf?svg=true)](https://ci.appveyor.com/project/nashwaan/xml-js)
[![Build Status](https://travis-ci.org/nashwaan/xml-js.svg?branch=master)](https://travis-ci.org/nashwaan/xml-js)
[![Build Status](https://img.shields.io/circleci/project/nashwaan/xml-js.svg)](https://circleci.com/gh/nashwaan/xml-js)

[![bitHound Overall Score](https://www.bithound.io/github/nashwaan/xml-js/badges/score.svg)](https://www.bithound.io/github/nashwaan/xml-js)
[![Coverage Status](https://coveralls.io/repos/github/nashwaan/xml-js/badge.svg?branch=master)](https://coveralls.io/github/nashwaan/xml-js?branch=master)
[![Code Climate](https://codeclimate.com/github/nashwaan/xml-js/badges/gpa.svg)](https://codeclimate.com/github/nashwaan/xml-js)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/f6ed5dd79a5b4041bfd2732963c4d09b)](https://www.codacy.com/app/ysf953/xml-js?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=nashwaan/xml-js&amp;utm_campaign=Badge_Grade)

[![npm](http://img.shields.io/npm/v/xml-js.svg)](https://www.npmjs.com/package/xml-js)
[![License](https://img.shields.io/npm/l/xml-js.svg)](LICENSE)
[![Dependency Status](https://david-dm.org/nashwaan/xml-js.svg)](https://david-dm.org/nashwaan/xml-js)
[![Package Quality](http://npm.packagequality.com/shield/xml-js.svg)](http://packagequality.com/#?package=xml-js)

# Synopsis

![Convert XML ↔ JS/JSON as compact or non-compact](http://nashwaan.github.io/xml-js/images/synopsis.svg)
<!---![Convert XML ↔ JS/JSON as compact or non-compact](/synopsis.png?raw=true "Synopsis Diagram")-->

# Motivation

There are many XML to JavaScript object / JSON converters out there, but could not satisfy the following requirements:

* **Maintain Order of Elements**:
Instead of converting `<a/><b/><a/>` to `{a:[{},{}],b:{}}`, I wanted to preserve order of elements by doing this: 
`{"elements":[{"type":"element","name":"a"},{"type":"element","name":"b"},{"type":"element","name":"a"}]}`.

* **Fully XML Compliant**:
Can parse: elements, attributes, texts, comments, CData, DOCTYPE, and XML declarations.

* **Reversible**:
Whether converting xml→json or json→xml, the result can be converted back to its original form.

* **Minimal Dependencies**:
This library depends only on one external npm module.

* **Change Property Key Name**:
Usually output of XML attributes are stored in `@attr`, `_atrr`, `$attr`, `$`, or `whatever` in order to avoid conflicting with name of sub-elements. 
This library store them in `attributes`, but most importantly, you can change this to whatever you like.

* **Support Upwards Traversal**:
By setting `{addParent: true}` option, an extra property named `parent` will be generated along each element so that its parent can be referenced.
Therefore, anywhere during the traversal of an element, its children **and** its parent can be easily accessed.

* **Portable Code**:
Written purely in JavaScript which means it can be used in Node environment and **browser** environment (via bundlers like browserify/JSPM/Webpack).

* **Support Command Line**:
To quickly convert xml or json files, this module can be installed globally or locally (i.e. use it as [script](https://docs.npmjs.com/misc/scripts) in package.json).

* **Typings Info Included**:
Support type checking and code suggestion via intellisense.
Thanks to the wonderful efforts by [DenisCarriere](https://github.com/DenisCarriere)
   
## Compact vs Non-Compact

Most XML to JSON convertors (including online convertors) convert `<a/>` to some compact output like `{"a":{}}` 
instead of non-compact output like `{"elements":[{"type":"element","name":"a"}]}`.

While compact output might work in most situations, there are cases when elements of different names are mixed inside a parent element. Lets use `<a x="1"/><b x="2"/><a x="3"/>` as an example.
Most converters will produce compact like `{a:[{_:{x:"1"}},{_:{x:"3"}}], b:{_:{x:"2"}}}`, 
which has merged both `<a>` elements into an array! If you try to convert this back to xml, you will get `<a x="1"/><a x="3"/><b x="2"/>` 
which has not preserved the order of elements! This is an inherit limitation in the compact representation 
because output like `{a:{_:{x:"1"}}, b:{_:{x:"2"}}, a:{_:{x:"3"}}}` is illegal (same property name `a` should not appear twice in an object).

The non-compact output, which is supported by this library, will produce more information and always gurantees the order of the elements as they appeared in the XML file.

Another drawback of compact output is the resultant element can be an object or an array and therefore makes the client code a little awkwards in terms of extra check of object type before processing.

NOTE: Although non-compact output is more accurate representation of original XML than compact version, the non-compact version is verbose and consumes more space. 
This library provides both options. Use `{compact: false}` if you are not sure because it preserves everything; 
otherwise use `{compact: true}` if you want to save space and you don't care about mixing elements of same type and loosing their order.

# Usage

## Installation

```
npm install --save xml-js
```

You can also install it globally to use it as a command line convertor (see [Command Line](#command-line)).

```
npm install --global xml-js
```

## Quick start

```js
var convert = require('xml-js');
var xml = 
'<?xml version="1.0" encoding="utf-8"?>' +
'<note importance="high" logged="true">' +
'    <title>Happy</title>' +
'    <todo>Work</todo>' +
'    <todo>Play</todo>' +
'</note>';
var result1 = convert.xml2json(xml, {compact: true, spaces: 4});
var result2 = convert.xml2json(xml, {compact: false, spaces: 4});
console.log(result1, '\n', result2);
```

To see the result of this code, see the output above in *Synopsis* section.

Or [run and edit](https://runkit.com/587874e079a2f60013c1f5ac/587874e079a2f60013c1f5ad) this code live in the browser.

## Sample Conversions

| XML | JS/JSON compact | JS/JSON non-compact |
|:----|:----------------|:--------------------|
| `<a/>` | `{"a":{}}` | `{"elements":[{"type":"element","name":"a"}]}` |
| `<a/><b/>` | `{"a":{},"b":{}}` | `{"elements":[{"type":"element","name":"a"},{"type":"element","name":"b"}]}` |
| `<a><b/></a>` | `{"a":{"b":{}}}` | `{"elements":[{"type":"element","name":"a","elements":[{"type":"element","name":"b"}]}]}` |
| `<a> Hi </a>` | `{"a":{"_text":" Hi "}}` | `{"elements":[{"type":"element","name":"a","elements":[{"type":"text","text":" Hi "}]}]}` |
| `<a x="1.234" y="It's"/>` | `{"a":{"_attributes":{"x":"1.234","y":"It's"}}}` | `{"elements":[{"type":"element","name":"a","attributes":{"x":"1.234","y":"It's"}}]}` |
| `<?xml?>` | `{"_declaration":{}}` | `{"declaration":{}}` |
| `<?xml version="1.0" encoding="utf-8"?>` | `{"_declaration":{"_attributes":{"version":"1.0","encoding":"utf-8"}}}` | `{"declaration":{"attributes":{"version":"1.0","encoding":"utf-8"}}}` |
| `<!--Hello, World!-->` | `{"_comment":"Hello, World!"}` | `{"elements":[{"type":"comment","comment":"Hello, World!"}]}` |
| `<![CDATA[<foo></bar>]]>` | `{"_cdata":"<foo></bar>"}` | `{"elements":[{"type":"cdata","cdata":"<foo></bar>"}]}` |

# API Reference

This library provides 4 functions: `js2xml()`, `json2xml()`, `xml2js()`, and `xml2json()`. Here are the usages for each one (see more details in the following sections):
```js
var convert = require('xml-js');
result = convert.js2xml(js, options);     // to convert javascript object to xml text
result = convert.json2xml(json, options); // to convert json text to xml text
result = convert.xml2js(xml, options);    // to convert xml text to javascript object
result = convert.xml2json(xml, options);  // to convert xml text to json text
```

## Convert JS object / JSON → XML

To convert JavaScript object to XML text, use `js2xml()`. To convert JSON text to XML text, use `json2xml()`.

```js
var convert = require('xml-js');
var json = require('fs').readFileSync('test.json', 'utf8');
var options = {ignoreText: true, spaces: 4};
var result = convert.json2xml(json, options);
console.log(result);
```

### Options for Converting JS object / JSON → XML

The below options are applicable for both `js2xml()` and `json2xml()` functions.

| Option                | Default | Description |
|:----------------------|:--------|:------------|
| `spaces`              | `0`     | Number of spaces to be used for indenting XML output. Passing characters like `'` &nbsp;&nbsp;&nbsp; `'` or `'\t'` are also accpeted. |
| `compact`             | `false` | Whether the *input* object is in compact form or not. |
| `fullTagEmptyElement` | `false` | Whether to produce element without sub-elements as full tag pairs `<a></a>` rather than self closing tag `<a/>`. |
| `indentCdata` | `false` | Whether to write CData in a new line and indent it. Will generate `<a>\n <![CDATA[foo]]></a>` instead of `<a><![CDATA[foo]]></a>`. |
| `ignoreDeclaration`   | `false` | Whether to ignore writing declaration directives of xml. For example, `<?xml?>` will be ignored. |
| `ignoreAttributes`    | `false` | Whether to ignore writing attributes of the elements. For example, `x="1"` in `<a x="1"></a>` will be ignored |
| `ignoreComment`       | `false` | Whether to ignore writing comments of the elements. That is, no `<!--  -->` will be generated. |
| `ignoreCdata`         | `false` | Whether to ignore writing CData of the elements. That is, no `<![CDATA[  ]]>` will be generated. |
| `ignoreDoctype`         | `false` | Whether to ignore writing Doctype of the elements. That is, no `<!DOCTYPE ]>` will be generated. |
| `ignoreText`          | `false` | Whether to ignore writing texts of the elements. For example, `hi` text in `<a>hi</a>` will be ignored. |

## Convert XML → JS object / JSON

To convert XML text to JavaScript object, use `xml2js()`. To convert XML text to JSON text, use `xml2json()`.

```js
var convert = require('xml-js');
var xml = require('fs').readFileSync('test.xml', 'utf8');
var options = {ignoreText: true, alwaysChildren: true};
var result = convert.xml2js(xml, options); // or convert.xml2json(xml, options)
console.log(result);
```

### Options for Converting XML → JS object / JSON

The below options are applicable for both `xml2js()` and `xml2json()` functions.

| Option              | Default | Description |
|:--------------------|:--------|:------------|
| `compact`           | `false` | Whether to produce detailed object or compact object. |
| `trim`              | `false` | Whether to trim white space characters that may exist before and after the text. |
| `sanitize`          | `false` | Whether to replace `&` `<` `>` `"` `'` with `&amp;` `&lt;` `&gt;` `&quot;` `&#39;` respectively in the resultant text. |
| `nativeType`        | `false` | Whether to attempt converting text of numerals or of boolean values to native type. For example, `"123"` will be `123` and `"true"` will be `true` |
| `addParent`         | `false` | Whether to add `parent` property in each element object that points to parent object. |
| `alwaysChildren`    | `false` | Whether to always generate `elements` property even when there are no actual sub elements (applicable for non-compact output). |
| `ignoreDeclaration` | `false` | Whether to ignore writing declaration property. That is, no `declaration` property will be generated. |
| `ignoreAttributes`  | `false` | Whether to ignore writing attributes of elements.That is, no `attributes` property will be generated. |
| `ignoreComment`     | `false` | Whether to ignore writing comments of the elements. That is, no `comment` will be generated. |
| `ignoreCdata`       | `false` | Whether to ignore writing CData of the elements. That is, no `cdata` will be generated. |
| `ignoreDoctype`     | `false` | Whether to ignore writing Doctype of the elements. That is, no `doctype` will be generated. |
| `ignoreText`        | `false` | Whether to ignore writing texts of the elements. That is, no `text` will be generated. |

The below option is applicable only for `xml2json()` function.

| Option              | Default | Description |
|:--------------------|:--------|:------------|
| `spaces`            | `0`     | Number of spaces to be used for indenting JSON output. Passing characters like `'` &nbsp;&nbsp;&nbsp; `'` or `'\t'` are also accpeted. |

## Options for Changing Key Names

To change default key names in the output object or the default key names assumed in the input JavaScript object / JSON, use the following options:

| Option              | Default | Description |
|:--------------------|:--------|:------------|
| `declarationKey`    | `"declaration"` or `"_declaration"` | Name of the property key which will be used for the declaration. For example, if `declarationKey: '$declaration'` then output of `<?xml?>` will be `{"$declaration":{}}` *(in compact form)* |
| `attributesKey`     | `"attributes"` or `"_attributes"` | Name of the property key which will be used for the attributes. For example, if `attributesKey: '$attributes'` then output of `<a x="hello"/>` will be `{"a":{$attributes:{"x":"hello"}}}` *(in compact form)* |
| `textKey`           | `"text"` or `"_text"` | Name of the property key which will be used for the text. For example, if `textKey: '$text'` then output of `<a>hi</a>` will be `{"a":{"$text":"Hi"}}` *(in compact form)* |
| `cdataKey`          | `"cdata"` or `"_cdata"` | Name of the property key which will be used for the cdata. For example, if `cdataKey: '$cdata'` then output of `<![CDATA[1 is < 2]]>` will be `{"$cdata":"1 is < 2"}` *(in compact form)* |
| `doctypeKey`          | `"doctype"` or `"_doctype"` | Name of the property key which will be used for the doctype. For example, if `doctypeKey: '$doctype'` then output of `<!DOCTYPE foo>` will be `{"$doctype":" foo}` *(in compact form)* |
| `commentKey`        | `"comment"` or `"_comment"` | Name of the property key which will be used for the comment. For example, if `commentKey: '$comment'` then output of `<!--note-->` will be `{"$comment":"note"}` *(in compact form)* |
| `parentKey`         | `"parent"` or `"_parent"` | Name of the property key which will be used for the parent. For example, if `parentKey: '$parent'` then output of `<a></b></a>` will be `{"a":{"b":{$parent:_points_to_a}}}` *(in compact form)* |
| `typeKey`           | `"type"` | Name of the property key which will be used for the type. For example, if `typeKey: '$type'` then output of `<a></a>` will be `{"elements":[{"$type":"element","name":"a","attributes":{}}]}` *(in non-compact form)* |
| `nameKey`           | `"name"` | Name of the property key which will be used for the name. For example, if `nameKey: '$name'` then output of `<a></a>` will be `{"elements":[{"type":"element","$name":"a","attributes":{}}]}` *(in non-compact form)* |
| `elementsKey`       | `"elements"` | Name of the property key which will be used for the elements. For example, if `elementsKey: '$elements'` then output of `<a></a>` will be `{"$elements":[{"type":"element","name":"a","attributes":{}}]}` *(in non-compact form)* |

Two default values mean the first is used for *non-compact* output and the second is for *compact* output.

> **TIP**: You probably want to set `{textKey: 'value', cdataKey: 'value', commentKey: 'value'}` for *non-compact* output
> to make it more consistent and easier for your client code to go through the contents of text, cdata, and comment.

# Command Line

Because any good library should support command line usage, this library is no difference.

## As Globally Accessible Command

```
npm install -g xml-js                       // install this library globally
xml-js test.json --spaces 4                 // xml result will be printed on screen
xml-js test.json --spaces 4 --out test.xml  // xml result will be saved to test.xml
xml-js test.xml --spaces 4                  // json result will be printed on screen
xml-js test.xml --spaces 4 --out test.json  // json result will be saved to test.json
```

## As Locally Accessible Command

If you want to use it as script in package.json (can also be helpful in [task automation via npm scripts](http://blog.keithcirkel.co.uk/how-to-use-npm-as-a-build-tool/))

```
npm install --save xml-js   // no need to install this library globally
```

In package.json, write a script:
```json
...
  "dependencies": {
    "xml-js": "latest"
  },
  "scripts": {
     "convert": "xml-js test.json --spaces 4"
  }
```
  
```
npm run convert             // task 'scripts.convert' will be executed
```

## CLI Arguments

```
Usage: xml-js src [options]

  src                  Input file that need to be converted.
                       Conversion type xml->json or json->xml will be inferred from file extension.

Options:
  --help, -h           Display this help content.
  --version, -v        Display version number of this module.
  --out                Output file where result should be written.
  --spaces             Specifies amount of space indentation in the output.
  --full-tag           XML elements will always be in <a></a> form.
  --no-decl            Declaration instruction <?xml ..?> will be ignored.
  --no-attr            Attributes of elements will be ignored.
  --no-text            Texts of elements will be ignored.
  --no-cdata           Cdata of elements will be ignored.
  --no-doctype         Doctype of elements will be ignored.
  --no-comment         Comments of elements will be ignored.
  --trim               Any whitespaces surrounding texts will be trimmed.
  --compact            JSON is in compact form.
  --sanitize           Special xml characters will be replaced with entity codes.
  --native-type        Numbers and boolean will be converted (coreced) to native type instead of text.
  --always-children    Every element will always contain sub-elements (applicable if --compact is not set).
  --text-key           To change the default 'text' key.
  --cdata-key          To change the default 'cdata' key.
  --doctype-key        To change the default 'doctype' key.
  --comment-key        To change the default 'comment' key.
  --attributes-key     To change the default 'attributes' key.
  --declaration-key    To change the default 'declaration' key.
  --type-key           To change the default 'type' key (applicable if --compact is not set).
  --name-key           To change the default 'name' key (applicable if --compact is not set).
  --elements-key       To change the default 'elements' key (applicable if --compact is not set).
```

# Contribution

## Testing

To perform tests on this project, download the full repository from GitHub (not from npm) and then do the following:

```
cd xml-js
npm install
npm test
```
For live testing, use `npm start` instead of `npm test`.

## Reporting

Use [this link](https://github.com/nashwaan/xml-js/issues) to report an issue or bug. Please include a sample code where the code is failing.

# License

[MIT](https://github.com/nashwaan/xml-js/blob/master/LICENSE)
---
title: 'File APIs（Blob, BlobURL, ArrayBuffer, FileReader）'
date: 2020-03-30 22:13:32
tags: [前端]
toc: true
---

## 前言

以前的JavaScript很不擅长处理二进制数据，要将二进制数据保存在浏览器上使用需要牺牲1.3倍的存储空间（将数据转换成 `DataURI` ）。即使是使用 `XHR` 从服务器请求数据，将读入内存之后还需要转换成 `Base64` / `DataURI` ），过程十分繁琐。

但是，时代已经变了。

现在的JavaScript中，处理二进制数据已经不是难题。硬件和 JIT（just-in-time）编译器的升级，让H5实现大多数API来处理大容量的二进制数据。

下面来介绍HTML5的Blob, File, FileList等对象，以及它们之间的相互转换。

## HTML5的File API和对象

### Blob

Blob可以更好的处理二进制数据和文件：

```javascript
class Blob {
    readonly size: UINT64 = 0,
    readonly type: String = "",

    constructor(source:AnyArray, option:BlobTypeObject = null):Blob { },
    slice(start:INT64 = 0, end:INT64 = 0, contentType:String = ""):Blob { },
    close():void {}
}
class BlobTypeObject {
    readonly type: String = ""
}
```

Blob具有 `size` 和 `type` 属性，并且还有 `slice` 和 `close` 方法，可以在 `ArrayBuffer ` 或 `BlobURL` 之间相互转换。

```javascript
var source = new Uint8Array([ 0,1,2 ]);
var blob = new Blob([ source ]);
var reader = new FileReader();

reader.onload = function() {
    var result = new Uint8Array(reader.result); // reader.result 是 ArrayBuffer
};
reader.readAsArrayBuffer(blob);
```

Blob构造器的第一参数指定一个数组，这个数组的元素类型可以是 `Blob` 、`ArrayBuffer` 、`TypedArray`、`String` 或 `Object` 。如果是 `Object` 的情况还需要用 `toString` 方法转换成字符串。如果有多个项的情况，将按照顺序链接Blob对象内容。

```javascript
// 创建一个空的Blob对象
var blob = new Blob();
```
Blob的第二个参数指定了 `{ type: MimeTypeString }`  ，`MimeTypeString ` 设置了Blob的种类 `MimeType` 。

```javascript
// 创建一个 octet-binary Blob 对象

var buffer1 = new Uint8Array(100);
var buffer2 = new Uint8Array(100);

var blob = new Blob([buffer1, buffer2], { type: "application/octet-binary" });
```

通常使用二进制数据时会指定类型，如： `{ type: "application/octet-binary" }` 。

```javascript
var blob = new Blob([file], { type: "image/png" });
var blob = new Blob([binary], { type: "application/octet-binary" });
var blob = new Blob(["字符串"], { type: "text/plain" });
var blob = new Blob(["<html>"], { type: "text/plain;charset=UTF-8" });
```

### File

File继承了Blob对象。并且追加了Blob的 `{ name:String, lastModifiedDate:Date }` 属性，File 是 JavaScript 中少数处理本地文件方法的一种。

```javascript
class File extends Blob {
    readonly name: String = "",
    readonly lastModifiedDate: Date = null,
    readonly webkitRelativePath: String = "",

    constructor(source:Blob, fileName:String):File { },
}
```

在Chrome中，File的构造器需要指定一个数组： `new File([], ...)` ，扩展了 `{ lastModified: Integer, webkitRelativePath: String }` 。

```javascript
// Chrome的示例
class File extends Blob {
    readonly name: String = "",
    readonly lastModified: Integer = 0,
    readonly lastModifiedDate: Date = null,
    readonly webkitRelativePath: String = "",

    constructor(source:AnyArray, fileName:String):File { },
}
```

由于File继承了Blob，因此File也支持使用 `FileReader.readAsArrayBuffer()` API转换为Blob。

```javascript
var reader = new FileReader();

reader.readAsArrayBuffer(file);
```

```javascript
function _toBlob(url, callback) {
    var xhr = new XMLHttpRequest();
    xhr.responseType = "blob";
    xhr.onload = function() {
        callback(xhr.response, url);
    };
    xhr.open("GET", url);
    xhr.send();
}

_toBlob(location.href, function(blob) {
    var blobURL = URL.createObjectURL(blob);
    var file = new File([blob], "foo.html"); // 或者是 new File(blob, ...)
});
```

### FileList

`FileList` 是通过DOM如 ` <input type="file" multiple="multiple">` 的 `onchange` 取得的File的列表。

```javascript
document.querySelector('input[type="file"]').addEventListener("change", function(event) {
    var fileList = event.target.files; // ArrayLikeObject

    for (var i = 0, iz = fileList.length; i < iz; ++i) {
        var file = fileList[0];
        ...
    }
});
```

```javascript
class FileList {
    readonly length: UINT32 = 0,

    item(index:UINT32):File { },
}
```

### FileReader

`FileReader` 类可以将二进制数据读入内存。

使用 `readAsArrayBuffer` 方法可以将 `Blob` 转换成 `ArrayBuffer` ，转换的结果通过 `result` 属性进行设置，同样可以使用 `readAsDataURL` 或 `readAsText` 等方法进行转换。

```javascript
class FileReader extends EventHandler {
    EMPTY:   UINT16 = 0,
    LOADING: UINT16 = 1,
    DONE:    UINT16 = 2,
    readonly error: Error = null,
    readonly result: ArrayBuffer|String = null,
    readonly readyState: UINT16 = EMPTY,
    onload:      EventHandler = null,
    onloadstart: EventHandler = null,
    onloadend:   EventHandler = null,
    onprogress:  EventHandler = null,
    onabort:     EventHandler = null,
    onerror:     EventHandler = null,

    // async read methods
    readAsArrayBuffer(blob:Blob): void {},
    readAsDataURL(blob:Blob):void {},
    readAsText(blob:Blob, label:String = ""):void {},

    abort():void {},
}
```

Chrome中还有 `readAsBinaryString(blob:Blob):void {}`  方法。

### BlobURL生成和释放

#### 生成

`BlobURL` 是 `blob:...` 的形式的假象URL，可以通过 `URL.createObjectURL(source:Blob):BlobURLString` 来动态生成：

```javascript
var uint8Array = new Uint8Array([1, 2, 3]);
var blob = new Blob([uint8Array], { type: "application/octet-binary" });
var blobURL = URL.createObjectURL(blob); // "blob:http%3A//dev.w3.org/207c851d-7486-42ec-97b1-d3cd26d08ed9"
```

`URL.createObjectURL` 总是生成唯一的 `BlobURL` （对于同一个资源每次都会生成不同的URL），生成的 `BlobURL` 在浏览器关闭之前有效。

#### 释放

如果要释放BlobURL，可以调用 `URL.revokeObjectURL(blobURL:BlobURLString):void` 方法。当然，也有 `URL.createFor(Blob):BlobURLString` 创建可以自动释放的BlobURL，不过要考虑兼容性。

```javascript
var blobURL = URL.createObjectURL(file);
// ...
URL.revokeObjectURL(blobURL);
```

`img.src` 、 `XMLHttpRequest.open(method, URL) ` 、`Audio` 等需要使用 URL 请求文件的情况，也可以通过生成BlobURL发送。

与DataURI时相比，使用BlobURL在速度和内存占用方面都更有优势，尤其是处理Audio等流式传输数据时，其差异会更明显。

```javascript
var img = document.createElement("img");
img.src = blobURL;
```

Chrome可以通过访问chrome://blob-internals/来确认有效的BlobURL。

## 相互转换

### Blob, File, BlobURL的相互转换

BlobURL可以通过XMLHttpRequest转换成Blob或者ArrayBuffer等其它类型。

```javascript
    var xhr = new XMLHttpRequest();
    xhr.onload = function() {
        var result = xhr.response; // ArrayBuffer
    };
    xhr.responseType = "arraybuffer";
    //xhr.responseType = "blob";
    xhr.open("GET", blobURL);
    xhr.send();
```

Blob,，BlobURLString， ArrayBuffer的相互转换可以通过以下方法：

| 原始      | 目标          | 方法                                        |
| :-------- | :------------ | :------------------------------------------ |
| Blob/File | BlobURLString | URL.createObjectURL(...)                    |
| BlobURL   | Blob          | XMLHttpRequest#responseType = "blob"        |
| BlobURL   | ArrayBuffer   | XMLHttpRequest#responseType = "arraybuffer" |
| Blob/File | ArrayBuffer   | FileReader#readAsArrayBuffer(...)           |
| Blob/File | BinaryString  | FileReader#readAsBinaryString(...)          |
| Blob/File | DataURLString | FileReader#readAsDataURL(...)               |
| Blob/File | String        | FileReader#readAsText(..., 编码方式)        |

转换ArryBuffer的方式可以如下：

```javascript
function _toArrayBuffer(source,     // @arg BlobURLString|URLString|Blob|File|TypedArray|ArrayBuffer
                        callback) { // @arg Function - callback(result:ArrayBuffer, source:Any):void
    if (source.buffer) {
        // TypedArray
        callback(source.buffer, source);
    } else if (source instanceof ArrayBuffer) {
        // ArrayBuffer
        callback(source, source);
    } else if (source instanceof Blob) {
        // Blob or File
        var reader = new FileReader();
        reader.onload = function() {
            callback(reader.result, source);
        };
        reader.readAsArrayBuffer(source);
    } else if (typeof source === "string") {
        // BlobURLString or URLString
        var xhr = new XMLHttpRequest();
        xhr.responseType = "arraybuffer";
        xhr.onload = function() {
            callback(xhr.response, source);
        };
        xhr.open("GET", source);
        xhr.send();
    } else {
        throw new TypeError("Unknown type");
    }
}
```

`FileReader#readAsText(blob, "UTF-8")` 第二个参数可以指定编码方式，默认为 "UTF-8" ：

```javascript
var reader = new FileReader();

reader.onload = function() {
    var buffer = reader.result; // String
};
reader.readAsText(file, "UTF-16");
```

### 参考链接

《File APIs(Blob, BlobURL, ArrayBuffer, FileReader)》：https://qiita.com/TypoScript/items/0d5b08cecf959b8b822c
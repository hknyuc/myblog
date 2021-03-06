﻿---
layout: post
title: "Weak (Gevşek) Referanslar"
comments: true
description: "Javascript'de bellek sızıntılarını önleyin."
keywords: "Weakset Weakmap map set"
---

Single Page Application (SPA) uygulamaların sayısı gün geçtikçe artıyor. Frontend tarafında kullanılan JS'e ait framework (çatı) ve library (kütüphane) sayısı gün geçtikçe artıyor, çeşitleniyor. SPA uygulamalarının sayısındaki artış, bizi bazı konularda daha dikkatli olmaya ve bazı problemlerin üzerinde daha çok durmaya zorluyor.

SPA uygulamalarının en büyük problemlerinden bir tanesi de bellek sızıntılarıdır (Memory Leaks).  JS kullanılmayan yapıları otomatik silen bir Garbage Collection (GC) mekanizmasına sahiptir. Herhangi bir yapı üzerinde referansı olmayan yapıları GC otomatik olarak silmektedir. Tarayıcıların da bellek yönetimi konusunda bazı teknikleri vardır. Bunlardan bir tanesi de çalışan sayfanın kapandıktan sonra bellek üzerinde kullandığı alanları silmesidir. SPA uygulamaları bir sayfa üzerinde çalıştıkları için bellek yönetimi konusunda daha hassas olmaları gerekir. Herhangi bir component kullanıldıktan sonra kullanılan scope içerisindeki referansların silinmesi gerekmektedir. JS'a ES6 ile gelen Weak Reference kavramı daha iyi bellek yönetimi konusunda yardımcı olmaktadır.

Weak Reference kavramı ilk olarak .Net Framework 4 de karşıma çıkmıştı. Javascript'e ise 2015 yılında ES6 ile geldi. Implementation ve kullanım biçiminde farklılık olsa da aşağı yukarı aynı işi yapmaktadır. JS'de **WeakMap** ve **WeakSet** collection yapıları ile bir referansı gevşetmemiz (weak) mümkün.

```js
let weakMap = new WeakMap();
let o1 = {},
    o2 = function () {},
    o3 = window,
    o4 = new String('value123'),
    o5 = 'key';

 weakMap.set(o1,37);
 weakMap.set(o2,'hakan');
 weakMap.set(o3,undefined);
 weakMap.set(o4,{name:'hakan'});
```

**WeakMap** bir hashtable. Set ettiğiniz her key'e sadece bir tane değer atayabilirsiniz. Kullanım biçim olarak Map ile çok büyük benzerlikleri var.

```js
weakMap.get(o1); // 37;
weakMap.get(o2); // hakan;
weakMap.get(o3); // undefined;
weakMap.get(o4); // {name:'hakan'};
```

**get** metodu girilen Key'e ait değer alınabiliyor. Dikkat ettiyseniz key olarak herhangi bir  referans tipli veri kullandık. Eğer değer tipli bir Key kullanırsak;

```js
weakMap.set(o5,'struct data type');
//Uncaught TypeError: Invalid value used as weak map key
```

__'Invalid value used as weak map key'__ hatası alıyoruz. Görüldüğü gibi Key'imiz sadece referans tipli olmak zorunda. Değer tipli veri yapıları doğaları gereği immutable yapıdadır. Kullanıldıkları yerin scope'u sonladığında bağlı oldukları referans tipinin silinmesi ile silineceklerdir. Bu yüzden Key alanına sadece referans tipli değerler alması zaten işimizi görmektedir.

```js
weakMap.has(o1); // true;
weakMap.has(o3); // true;
weakMap.has(o5); // false;
```

**has** metodu ile girilen Key'in collection içerisinde olup olmadığını kontrol edebiliriz. Dikkat ettiyseniz değeri undefined olarak girilse bile metot sonucu true olarak dönmektedir. İçerisindeki herhangi bir değeri silmek için **delete** metodunu kullanabilirsiniz.

```js
weakMap.delete(o3);
weakMap.has(o3); // false;
```

Eğer set ettiğimiz referansı null olarak atarsak, **WeakMap** içerisindeki değerin silindiğini göreceğiz.

```js
o1 = null;
weakMap.has(o1) // false;
```

**WeakMap** sınıfı iterasyonu desteklememektedir.

```js
for(let i of weakMap)
    console.log(i);
// TypeError: weakMap is not iterable
```

Bunun nedeni ise GC'in silme işlemini asenkron olarak yapması (non-deterministic algorithms) ve silme işleminden WeakMap sınıfının haberinin olmamasıdır.

Peki bize HashTable değil de sadece Collection lazım olduğunda ne yapmamız gerekiyor? **WeakSet** imdadımıza yetişiyor.

```js
let weakSet = new WeakSet();
let book = {};
let paper = {};

weakSet.add(book);
weakSet.add(paper);

weakSet.has(book); // true;
weakSet.has(paper);// true;

weakSet.delete(book); // true;
weakSet.has(book); // false;
weakSet.has(paper); // true;
paper = null; 
weakSet.has(paper); // false;
```

**WeakMap**'de olduğu gibi **WeakSet** de iterasyonu desteklememektedir. Peki iterasyon olmayan bir set implementasyonu ne işimize yarayabilir? Kullanım alanı tabiki normal Set collection'a göre az olmasına rağmen yine de önemli kullanım alanları mevcut. Bir tane örnek vermek gerekirse;

```js
// original code at :https://esdiscuss.org/topic/actual-weakset-use-cases#content-1
// thanks Domenic Denicola.
const foos = new WeakSet();
class Foo {
  constructor() {
    foos.add(this);
  }
  
  method() {
    if (!foos.has(this)) {
      throw new TypeError("Foo.prototype.method called on an incompatible object!");
    }
  }
}
```

New instance alınmadan cağrılan Foo Class'ınını kullanıma kapattık. Eğer bunu normal Set collectini ile yapsaydık, alınan bütün Foo instancelarını globaldaki Scope'un içine alarak hafızayı şişirmiş olacaktık.

---

Bu yazımda aşağıdaki sayfalardan da yararlandım;
1. [Using the WeakSet object](https://developer.mozilla.org/tr/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)
2. [Using the WeakMap object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
3. [Domenic Denicola's Weak Code](https://esdiscuss.org/topic/actual-weakset-use-cases#content-1)
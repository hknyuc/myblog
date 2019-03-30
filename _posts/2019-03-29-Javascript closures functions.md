---
layout: post
title: "Javascript Closures Function"
comments: true
description: "Javascript closures kavramı; bakımı kolay, temiz fonksiyonlar"
keywords: "Js Javascript Closures Function Clean-Code"
---

**Closures**, JS geliştiricileri tarafından kafa karışıklığına neden olan kavramların başında geliyor. Tabiki bu kavram sadece JS'e ait değil. Çoğu dilde mevcut. Bu yazım JS'de closures (kapatılmış) fonksiyon geliştirme tekniği üzerine olacak.

# Scope Nedir

**Closures** kavramını açıklamadan önce **Scope** kavramına değinmemiz gerekiyor. Scope kendi içinde ikiye ayrılmaktadır;

1. Local Scope
2. Global Scope

## Local Scope

Eğer bir değişkenimizi bir fonksiyon içerisinde tanımlıyorsak buna **Local Scope** denir.

```js
function myFunctionExample (){
    var myVariable = "car";
    console.log(myVariable + ":"+myVariable);
}

myFunctionExample();
//output: car:car
console.log(myVariable);
//output: ReferenceError: myVariable is not defined
```

Yukarıdaki örnekten de göreceğiniz üzere, eğer bir fonksiyon içerisinde bir değişken tanımladıysak, o değişkene sadece o fonksiyon üzerinden erişebiliyoruz. **myVariable**, **myFunctionExample** fonskiyonuna göre **Local Scope** içerisindedir diyebiliriz.

## Global Scope

Değişkenimiz kullanıldığı fonksiyonun dışında tanımlandıysa buna **Global Scope** denir.

```js
var myVariable = "car";
function myFunctionExample(){
    console.log(myVaribale);
}

muFunctionExample();
//output : car
console.log(myVariable);
//output : car
```

Gördüğünüz üzere **MyFunctionExample** içinde tanımlanmayan bir değişken, fonksiyon içerisinde kullanılmış. **myVariable**, **myFunctionExample** fonksiyonuna göre **Global Scope** içerisindedir diyebiliriz.

# JS değişkenlerinin yaşam süresi (lifetime)

Bir değişken tanımladığımızda doğal olarak bu değişkene **memory** üzerinden bir alan tahsil ediliyor. **Local Scope** içinde tanımladığımız değişkenler, fonksiyon çalıştırıldıktan sonra hafızadan otomatik olarak silinmektedir. Fakat **Global Scope** içindeki değişkenin silinmesi için bağlı bulunduğu fonksiyonun çalıştırmayı sonlandırması gerekiyor. Bu yüzden değişken tanımlarken Local mı, Global mi olması gerektiğine dikkat etmemiz gerekiyor. Global tanımlayacağımız değişkenlerin  hafıza üzerinden sızıntılar oluşturmaması için, dikkat edilmesi gereken bazı teknikler mevcut. Ayrıca [Weak Referanslar]({% post_url 2019-03-12-Weak (Gevşek) Referanslar %}) başlıklı yazımda bunlara değindim, göz atabilirsiniz.

# Let / Var / Const Kullanımı

**ECMAScript 2015** ile beraber **let**,**const** ile de değişken tanımlama özelliği eklendi. Dikkat ettiyseniz yukarıda kullandığımız bütün örneklerde **var** kullanarak değişken tanımladık. Bu iki kavram **Local Scope** içerisinde daha kontrol edilebilir ve temiz kod yazmamıza yardımcı oluyor.

Yazdığımız bütün örneklerde Scope'un başlangıç ve bitişini fonksiyonun sınırları belirliyordu. Fakat **ECMAScript 2015** ile birlikte **JavaScript Block Scope** dediğimiz kavram geldi. Örnek vermek gerekirse;

```js
 function myFunction(){
     {
        var x = 2;
     }
     console.log(x);
 }

 myFunction();
//output: 2
```

 **JavaScript Block Scope** ile birlikte scope'un sınırlarını **{}** blockları belirlemiş oldu. **var** ile tanımladığımız değişkene dışarıdan erişebildik.

 ```js
 function myFunction(){

     {
         let x = 2;
     }
     console.log(x);
 }
 myFunction();
 //output: ReferenceError: x is not defined
 ```

**let** ile tanımladığımız değişkenimize **{}** dışından erişmeye çalıştığımızda hata alıyoruz. Sadece **Local Scope** içerisinde erişime açık olmuş oluyor.

```js
function myFunction (){
    var x=5;
    let y=5;
    {
        var x = 10;
        let y = 10;
    }
    console.log("x:"+x);
    console.log("y:"+y);
}

myFunction();

//output: x:10
//output: y:5
```

Görüldüğü üzere **let** ile **Local Scope** içinde yaptığımız değişiklikler sadece o Scope içerisinde kalmaktadır.  

**const** kullanım biçimi olarak **let** ile aynı fakat, **const** ile tanımlanmış bir değişkenin referansını sonradan değiştiremiyoruz.

```js
  const myVariable = "this variable is constant";
  myVariable = "";
  //output : TypeError: Assignment to constant variable
```

**const** olarak tanımlamış olduğum **myVariable** değişkenine yeni bir değer atamak istediğimde hata aldım. Siz de tanımladığınız değişkenin değerinin sonradan değişmesini istemiyorsanız bunu kullanabilirsiniz.

## Closures Kavramı

Bir fonksiyonun **Closures** olması, o fonksiyonun kullandığı değişkenlerin dışarıdan erişilememesi anlamına gelmektedir. Kendi içerisinde kapalı (**Closures**) olması demektir. Örnek üzerinden gidersek daha anlaşılır olacağını düşünüyorum.

```js
var cars = ["BMW","Ford","Fiat"];
function getMyCars(){
    return {
        show:function (){
            console.log(cars.join(','));
            return cars.join(',');
        },
        remove:function (car){
            let indexOfItem = cars.indexOf(car);
            if(indexOfItem === -1) throw new Error(car + ' could not found');
            cars.splice(indexOfItem,1);
        }
    }
}

let myCars = getMyCars();
myCars.show();
// output: BMW,Ford,Fiat
myCars.remove('Fiat');
myCars.show();
// output: BMW,Ford
cars = []; // is manipulated from another process
myCars.show();
// output:
```

Yukarıdaki örnekte **getMyCars** adında bir fonksiyon tanımladık. Fonksiyon çalıştırıldığında bize **show** ve **remove** adında iki metodu olan bir obje dönüyor. Objemizin metotları kararlı bir şekilde çalışmıyor. Çünkü beslendiği verinin kaynağının referansı **global scope**'ta bulunmakta. Ve aynı scope içerisinde başka bir kod parçası bu değişkeni manipüle edebiliyor. Bu özelliklerden dolayı **getMyCars** fonksiyonu **Closures** özelliği taşımıyor diyebiliriz. Peki kodumuzu nasıl **Closures** yapabiliriz ?

```js
  function getMyCars(){
      return (function (){
          const cars = ["BMW","Ford","Fiat"];
          return {
             show:function (){
                 console.log(cars.join(','));
                 return cars.join(',');
             },
             remove:function (car){
                 let indexOfItem = cars.indexOf(car);
                 if(indexOfItem === -1) throw new Error(car + ' could not found');
                 cars.splice(indexOfItem,1);
             }
          }
      })();
  }

  let myCars = getMyCars();
  myCars.show();
  //output : "BMW,Ford,Fiat"
  myCars.remove('Fiat');
  myCars.show();
  //output : "BMW,Ford";
```

Yazdığımız fonksiyonda **cars** değişkeni içerideki bir fonksiyon içerisinde tanımladığımız için gerçek anlamada **private** bir değişken tanımlamış olduk ve bu değişkenimizi dışarıdan erişime kapattık. **Closures** kullanımı sayesinde **Object-Oriented Programming** yönelime ait **data-hiding**, **encapsulation** gibi kavramları sağlamış oluyoruz.

## Döngü içinde Closures yapılar oluşturmak

```js
function runExample(){
    for(var i=0;i<4;i++){
        setTimeout(()=>{
            console.log(i);
        },i*1000);
    }
}
runExample();
//output : 4,4,4,4
```

**runExample** fonksiyonunu içerisinde bir döngü tanımladık, döngüde iterator index olarak tanımladığımız **i** değişkenini **asenkron** olarak bu değişkeni ekrana yazdırdık. Beklediğimiz değer **1,2,3,4** olması gerekiyorken sonuç **4,4,4,4** oldu. Bunun nedeni **setTimeout** fonkisyonuna parametre olarak gönderdiğimiz fonksiyonun **i** değişkeninin referansını scopuna almasıdır. Iteratorun değer ataması **1000 ms** yani **1 saniyeden** daha az bir sürede olduğu için, **i** değişkenine atılan son değer 4 oldu. Tanımlamış olduğumuz bütün fonksiyonlar 4 değerini ekrana yazmış oldular. Peki bu problemi nasıl çözebiliriz?

```js
function runExample(){
    for(var i=0;i<4;i++){
        (function (a) {
            setTimeout(()=>{
            console.log(a);
            },a*1000);
       })(i);
    }
}
runExample();
//output: 0
//output: 1
//output: 2
//output: 3
```

Şimdi oldu, yazdığımız fonksiyonu başka bir fonksiyon içerisine alıp en üstteki fonksiyona değerimizi parametre olarak gönderdik. Peki bu nasıl çalıştı? **index** olarak belirlediğimiz **i** değişkeni değer tipli bir değişkendir, değer tipli değişkenler fonksiyonlara parametre olarak gönderildiklerinde, içindeki değerler kopyalanarak yeni bir değişken olarak tanımlanır. Bu da fonksiyonumuzun **closures** olduğu anlamına gelmektedir. Bu problemi **i** değişkenimizi **let** ile tanımlayarak da çözebiliriz.

```js
function runExample(){
    for(let i=0;i<4;i++){
        setTimeout(()=>{
            console.log(i);
        },i*1000);
    }
}

runExample();
//output: 0
//output: 1
//output: 2
//output: 3

```

For iterasyonu içerisinde **let** tanımlaması ile tek bir değişken yerine, her döngüde yeni bir değişken  tanımlamaktadır.

## Closures Neden Önemli?

**Functional Programming** paradigmaları popülerliği her geçen gün artırmakta. JS tam anlamıyla bir functional bir dil olmasa da çoğu özelliğini içerisinde barındırıyor. İç içe girmiş, aynı scope'u paylaşan, çalıştırıldıklarında da fonksiyon döndüren fonksiyonların bakımı, okunabilirliği zorlaşıyor.  **Closures** daha okunabilir, test edilebilir, yeniden kullanılabilir, hafızada sızıntı oluşturmayan kod yazmamızı sağlamaktadır.

---

Bu yazımda aşağıdaki sayfalardan da yararlandım;

1. [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

2. [https://www.w3schools.com/js/js_function_closures.asp](https://www.w3schools.com/js/js_function_closures.asp)

3. [https://www.quora.com/Why-are-closures-important-in-JavaScript](https://www.quora.com/Why-are-closures-important-in-JavaScript)
---
layout: post
title: "Javascript ve closure kavramı"
comments: true
description: ""
keywords: ""
---


**Closure**, JS geliştiricileri tarafından kafa karışıklığını neden olan kavramlarından başında geliyor. Tabiki bu kavram sadece JS'e ait bir kavram değil. Çoğu dilde bu kavram mevcut. Bu yazımda JS'de closure yazılım geliştirme tekniği üzerine olacak. Basit ve açıklayıcı yazmaya özen göstereceğim. 

# Scope Nedir

**Closure** kavramını açıklamadan önce **Scope** kavramına değinmemiz gerekiyor. Scope kendi içinde ikiye ayrılmaktadır;

1. Local Scope
2. Global Scope

## Local Scope
Eğer bir değişkenimiz bir fonksiyon içerisinde tanımlıyorsanız buna **Local Scope** denir.

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

Örnektenden göreceğiniz üzere, eğer bir fonksiyon içerisinde bir değişken tanımladıysak o değişkene sadece o fonksiyon üzeriden erişebiliyoruz. **myVariable**, **myFunctionExample** fonskiyonuna göre **Local Scope** içerisindedir diyebiliriz.

## Global Scope

Eğer tanımladığımız değişken, kullanıldığı fonksiyonun dışında tanımlandıysa buna **Global Scope** denir.

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

# Js değişkenlerinin yaşam süresi (lifetime)
Bir değişken tanımladığınızda doğal olarak bu değişkene **memory** üzerinden bir alan tahsil ediliyor. **Local Scope** içinde tanımladığınız değişkenler fonksiyon çalıştırılıktan sonra hafızadan otomatik olarak silinmektedir. Fakat **Global Scope** içindeki değişkenin silinmesi için bağlı bulunduğu fonksiyonun çalıştırmayı sonlandırması gerekiyor. Bu yüzden değişken tanımlarken Local mı, Global olması gerektiğini dikkat etmek gerekiyor. Global tanımlayacağınız değişkenleri de hafıza üzerinden sızıntılar oluşmaması için dikkat edilmesi gerek bazı teknikler mevcut, [Weak Referanslar]({% post_url 2019-03-12-Weak (Gevşek) Referanslar %}) yazımda bunlara değindim, okuyabilirsiniz. 

# Let / Var / Const Kullanımı

**ECMAScript 2015** ile beraber **let**,**const** ile de değişken tanımlama özelliği eklendi .Dikkat ettiyseniz yukarıda kullandığımız bütün örneklerde **var** kullanarak değişken tanımladık. Bu iki kavram **Local Scope** içerisinde daha kontrol edilebilir, temiz kod yazmamıza yarıyor.

Yazdığımız bütün örneklerde Scope'un başlangıç ve bitişini fonksiyonun sınırıları belirliyordu. Fakat **ECMAScript 2015** ile birlikte **JavaScript Block Scope** dediğimiz kavram geldi. Örnek vermek gerekirse;

```js
 function myFunction(){
     {
         var x = 2;
     }
     console.log(x);
 }

 myFunction();
//output: 2;
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
**let** ile tanımladığımız değişkenimize **{}** dışından erişmeye çalıştığımızda hata alıyoruz. Sadece **Local Scope** içerisinde erişme açık olmuş oluyor. **const** kullanım biçimi olarak **let** ile aynı fakat, **const** ile tanımlanmış bir değişkenin referansını sonradan değiştiremiyoruz.

```js
  const myVariable = "this variable is constant";
  myVariable = "";
  //output : TypeError: Assignment to constant variable
```
**const** olarak tanımlamış olduğum **myVariable** değişkenine yeni bir değer atamak istediğimde hata aldım. Tanımladığınız değişkenin değerinin sonradan değişmesini istemiyorsanız kullanabilirsiniz.


## Closure Kavramı

Bir fonksiyonun **Closure** olması, o fonksiyonun kullandığı değişkenlerin dışarıdan erişilememesi anlamına gelmektedir. Kendi içerisinde kapalı (**Closure**) olması demektir. Örnek üzerinden gidersek daha anlaşılır olacağını düşünüyorum. 

```js
const cars = ["BMW","Ford","Fiat"];
function getMyCars(){
    return {
        show:function (){
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
myCars.show(); // returns BMW,Ford,Fiat

```


```js
  function getMyCars(){
      return (function (){
          const cars = ["BMW","Ford","Fiat"];
          return {
             show:function (){
                 return cars.join(',');
             },
             delete:function (car){
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
  myCars.delete('Fiat');
  myCars.show();
  //output : "BMW,Ford";
```






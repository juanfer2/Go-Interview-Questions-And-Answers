
 <h2 dir="rtl"> 🌱Q1: چه تایپ هایی مقدار zero آن ها nil هست؟</h2>

* interfaces
* slices
* channels
* maps
* pointers  
* functions

---

 <h2 dir="rtl"> 🌱Q2: تایپ های نوع Reference؟</h2>  

* Pointers
* slices
* maps
* functions
* channels

---

 <h2 dir="rtl"> 🌱Q3: تایپ های نوع Aggregate؟</h2>  

* Array 
* structs

---

<h2 dir="rtl">🌱Q4: چه وقت باید از پوینتر استفاده کنیم؟</h2>
 <p dir="rtl">
1- تابعی که یکی از پارامترهای خود را تغییر می‌دهد
<br>
-وقتی تابعی را فراخوانی می‌کنیم که یک پوینتر را به عنوان پارامتر می‌گیرد، انتظار داریم که متغیر ما تغییر داده شود. اگر شما متغیر را در تابع خود تغییر نمی‌دهید، پس احتمالا نباید از پوینتر استفاده کنید.
 </p>
 <p dir="rtl">
2- عملکرد بهتر
<br>
-اگر رشته‌ای داشته باشید که شامل یک رمان کامل در حافظه باشد، کپی کردن این متغیر هر بار که به یک تابع جدید ارسال می‌شود، کاری بسیار گران است. ممکن است ارزشمند باشد که به جای این کار یک پوینتر را ارسال کنید، که باعث صرفه‌جویی در پردازنده و حافظه می‌شود. با این حال انجام این کار به قیمت خوانا بودن است، بنابراین فقط در صورت لزوم این بهینه‌سازی را انجام دهید.
  </p>
 <p dir="rtl">
3- به گزینه nil نیاز دارید
<br>
-گاهی اوقات یک تابع باید بداند که مقدار یک چیزی چیست، همچنین باید وجود یا عدم وجود آن را بداند. معمولا هنگام خواندن JSON از این استفاده می‌کنیم تا بدانیم فیلدی وجود دارد یا خیر.
 </p>

---

 <h2 dir="rtl"> 🌱Q5: زبان گولنگ از موارد زیر پشتیبانی نمی کند</h2>

* type inheritance
* operator overloading
* method overloading
* pointer arithmetic
* struct type in consts

---
 <h2  dir="rtl"> 🌱Q7: چه موقعی از channel و چه موقعی از mutex استفاده میشه برای گورتینگ ها؟(بحث ارتباط)؟ </h2>  
 <p  dir="rtl">
معمولاً در مواقعی که Goroutines نیاز به برقراری ارتباط با یکدیگر دارند از channels  استفاده کنید 
و درصورتی که فقط یک Goroutine دارید و باید به بخش مهم کد دسترسی داشته باشد از Mutexes استفاده کنید.
 </p>

---

 <h2  dir="rtl"> 🌱Q8: چرا کپی کردن pointer کند تر از کپی کردن مقدار هست؟</h2>  
 <p   dir="rtl">
- برای ارسال مقادیر کوچیکی که به مقدارشون فقط نیاز داریم از پوینتر استفاده نکنیم. <br>
- توی متغیرهای کوچیک (کمتر از ۳۲کیلوبایت) کپی کردن یک پوینتر تقریبا به اندازه کپی کردن مقدار اون متغیر هزینه داره  پس از این جهت سودی نمیبریم.<br>
- کامپایلر چک هایی رو تولید میکنه که موقع ران‌تایم زمان dereferencing پوینتر اجرا میشن.<br>
- پوینتر ها اکثرا توی Heap ذخیره میشن<br>
- برای این کار از ابزار های Go استفاده میکنیم ( go build -gcflags="-m" main.go )<br>
- اما اگر به صورت مقداری برگردونیم در stack ذخیره میشه.<br>
- همونطوری که میدونیم ذخیره در stack بسیار بهینه تر هست.<br>
- درواقع Garbage collector میاد heap رو چک میکنه و همونطوری که میدونیم هربار GC درحال بررسی هست به مدت چند میلی‌ثانیه کل سرویس ما فریز میشه. و میتونه مشکل هایی مثل Memory Leak و .. بوجود بیاد<br>
 </p>

---

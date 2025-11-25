# Report-7-2


---

# گزارش پروژه: ارتباط I2C بین دو آردوینو (Master & Slave Communication)

## ۱. مقدمه  
در این پروژه، دو **برد آردوینو** به هم متصل شده‌اند و با استفاده از پروتکل **I2C**، داده‌ها را بین آن‌ها ارسال و دریافت می‌کنند. یک آردوینو به عنوان **Master** کار می‌کند و دیگری به عنوان **Slave**.

## ۲. اهداف پروژه  
- آشنایی با پروتکل I2C و نحوه‌ی کار آن  
- یادگیری نحوه‌ی اتصال دو آردوینو به هم  
- کنترل یک آردوینو (Master) بر روی دیگری (Slave)  
- ارسال و دریافت داده‌ها بین دو آردوینو  
- آشنایی با مفهوم “Master” و “Slave”

## ۳. قطعات مورد استفاده  
- دو برد آردوینو Uno  
- برد برد (Breadboard)  
- سیم‌های جامپر (jumper wires)



## ۴. نحوه‌ی ساخت مدار  
- **اتصال I2C**:  
  - SDA → SDA (پین A4 آردوینو ۱ به پین A4 آردوینو ۲)  
  - SCL → SCL (پین A5 آردوینو ۱ به پین A5 آردوینو ۲)  
  - GND → GND (زمین دو آردوینو به هم متصل شود)  
  - VCC → VCC (ولتاژ دو آردوینو به هم متصل شود — اگر از منبع تغذیه جداگانه استفاده می‌کنید)



## ۵. کد برنامه (اسکچ)

### کد Master:
```cpp
#include <Wire.h>   // اضافه کردن کتابخانه I2C

int value = 0;

void setup() {
  Wire.begin();        // شروع I2C به عنوان Master
  Serial.begin(9600);  // شروع ارتباط سریال برای نمایش داده
  pinMode(5, OUTPUT);  // تنظیم پین ۵ به عنوان خروجی
}

void loop() {
  int y = analogRead(A1);          // خواندن مقدار آنالوگ از A1
  value = map(y, 0, 1023, 0, 255); // تبدیل محدوده

  Wire.beginTransmission(8);         // شروع ارسال به دستگاه #8 (Slave)
  Wire.write("Master value : ");    // ارسال متن
  Wire.write(value);                // ارسال مقدار
  Wire.endTransmission();           // پایان ارسال

  Wire.requestFrom(8, 1);           // درخواست ۱ بایت از Slave
  while (Wire.available()) {        // تا زمانی که داده موجود باشد
    int c = Wire.read();            // دریافت داده
    analogWrite(5, c);              // تنظیم شدت نور LED
    Serial.print("Slave value : ");
    Serial.println(c);              // نمایش داده در Serial Monitor
  }

  delay(100);  // تأخیر کوتاه
}
```

### کد Slave:
```cpp
#include <Wire.h>   // اضافه کردن کتابخانه I2C

int x;
int value = 0;

void setup() {
  Wire.begin(8);             // شروع I2C به عنوان Slave با آدرس ۸
  Wire.onRequest(requestEvent); // ثبت تابع برای درخواست داده
  Wire.onReceive(receiveEvent); // ثبت تابع برای دریافت داده
  Serial.begin(9600);          // شروع ارتباط سریال برای نمایش داده
  pinMode(9, OUTPUT);          // تنظیم پین ۹ به عنوان خروجی
}

void loop() {
  delay(100);
  analogWrite(9, x);           // تنظیم شدت نور LED

  int t = analogRead(A0);      // خواندن مقدار آنالوگ از A0
  value = map(t, 0, 1023, 0, 255); // تبدیل محدوده
}

// تابعی که هنگام دریافت داده از Master اجرا می‌شود
void receiveEvent() {
  while (1 < Wire.available()) { // تا زمانی که بیش از ۱ بایت داده موجود باشد
    char c = Wire.read();        // دریافت داده به صورت کاراکتر
    Serial.print(c);             // نمایش کاراکتر
  }
  x = Wire.read();               // دریافت داده به صورت عدد صحیح
  Serial.println(x);             // نمایش عدد صحیح
}

// تابعی که هنگام درخواست داده از Master اجرا می‌شود
void requestEvent() {
  Wire.write(value);             // ارسال مقدار به Master
}
```

### توضیح کد:  
- `Wire.begin()` — شروع I2C به عنوان Master.  
- `Wire.begin(8)` — شروع I2C به عنوان Slave با آدرس ۸.  
- `Wire.beginTransmission(8)` — شروع ارسال به دستگاه #8 (Slave).  
- `Wire.write()` — ارسال داده.  
- `Wire.endTransmission()` — پایان ارسال.  
- `Wire.requestFrom(8, 1)` — درخواست ۱ بایت از Slave.  
- `Wire.read()` — دریافت داده.  
- `analogWrite(5, c)` — تنظیم شدت نور LED بر اساس داده دریافتی.  
- `delay(100)` — برای اینکه نمایش در Serial Monitor قابل خواندن باشد.

## ۶. نحوه‌ی کارکرد  
- پس از آپلود کد، Master شروع به ارسال داده به Slave می‌کند.  
- Slave داده را دریافت کرده و بر اساس آن، شدت نور LED را تنظیم می‌کند.  
- Slave همچنین داده‌ای را به Master ارسال می‌کند و Master آن را دریافت کرده و نمایش می‌دهد.  
- این الگو به صورت بی‌پایان تکرار می‌شود.

## ۷. نتایج  
دو آردوینو به درستی با هم ارتباط برقرار می‌کنند. این نشان‌دهنده‌ی عملکرد صحیح ارتباط I2C است.

เมื่อพูดถึง microservice แต่ละที่นิยามไม่เหมือนกัน ไม่ได้มี standard กลาง 
ถ้าจะคุยเรื่องที่ให้เอาภาพมากาง


![Microservice Pattern](https://microservices.io/i/MicroservicePatternLanguage.jpg)

Testing ใน microservice มีอยู่ 3 เรื่อง 
- consumer-driven contract test
- consumer-side contract test
- service component test

แต่ก็ไม่ได้มีใครให้นิยามไว้ ดังนั้นก่อนทำต้องมานิยามของเราเองกันก่อน

ย้อนกลับมาที่ Test strategy ที่ martinfowler ใส่ไว้
ใน 1 service จะประกอบไปด้วย layer ต่างๆ ดังนี้
ซึ่งเราต้องทดสอบ service นั้น

1.Unit Test 

ปกติเรามองภาพรวมก่อนถึงจะมาหาปัญหา แต่จริงๆ เราต้องแก้ปัญหาในส่วนย่อยๆ ตั้งแต่แรก 
เหมือนในโรงงานอุตสาหกรรมที่สร้างตำแหน่ง QC มาเพื่อหาปัญหา แต่สุดท้ายแก้ไปเรื่อยๆ ก็ได้แต่ปัญหาปลายทาง ไม่ได้แก้ที่ Root Cause

ดังนั้นการทดสอบในหน่วยย่อยๆ สำคัญ 

แยกออกเป็น 2 ประเภทคือ 
- Solitary ทำงานคนเดียวโดดๆ
- Sociable สายคุยกับชาวบ้าน

เล็กสุด และเร็วที่สุด ต้องมีเยอะที่สุด แต่ตอบไม่ได้การันตีได้ว่าระบบทำงานด้วยกันได้

```
Domain คือชื่อในกลุ่มนั้นๆ เก็บ data ตาม aggegrate เป็นหลัก ไม่ได้มี Business Logic (อยู่ใน service)
```
2.Integration Testing
ของ martinfowler คือการเชื่อมส่วนภายนอกที่ควบคุมไม่ได้ (db and third party) `ต้องเชื่อมกับของจริง` เนื่องจากส่วนใหญ่องค์กรติดปัญหาเรื่อง communication 
ควรจะ integration test ก่อนที่เราจะ deploy ไปใช้ แต่คนส่วนใหญ่ชอบ deploy ขึ้นไปก่อนแล้ว integration test (readiness check)

integration test -> คือการเช็ค readiness ของ service
```
ที่ทำอยู่ไม่ใช่ integration test เป็น Component Test
```

3. Component Test
ทดสอบว่าระบบสามารถทำงานได้ ถ้าตัด third party ออกไป
ปัญหาคือเราไม่รู้ว่าของจริงเป็นอย่างไร เพราะเรา stub, จำลอง ไว้

ซึ่งการันตีไม่ได้ถ้า External Service เปลี่ยน
ปกติเวลาเราทดสอบคือเราเป็น client ไปทดสอบ external service 
แต่กลับกันถ้า external service เปลี่ยน มีใครมาทดสอบไหมว่า client พังไหม
เป็นแบบ reactive ซึ่งเราควรเปลี่ยน เพราะถ้ายิ่งเรารู้ช้าว่า external service เปลี่ยนแล้วทำให้ service พัง

ดังนั้นเค้าจึงหาวิธีการแก้ของ 2 ฝั่งที่ไม่ค่อยมี Communication ร่วมกัน
- คนให้บริการต้องรู้ว่า มีใครใช้งานอยู่บ้าง (เช่น open API)
- client ไม่รู้ว่า stub ไม่ update 

4. Contract Test
เมื่อพูดถึง Contract เราจะเคยได้ยินชื่อ Design by Contract
Caller --> Callee
```java
class Caller {
    void process() {
        Callee callee = new Callee();
        callee.doSomething(); // < -- feedback เร็วว่าถ้า parameter เปลี่ยนจะทำให้อย่างอื่นพัง
        
    }
}
class Callee {
    void doSomething(String data) {

    }
}
```
แต่ถ้าเป็นแบบนี้จะทำให้ทุกอย่างพัง เนื่องจาก callee ถูก coupled 
วิธีที่แก้คือสร้าง interface มาเป็น contract แทนให้ไม่ผูกกัน
```java
class Caller {
    void process() {
        Callee callee = new Callee();
        callee.doSomething();
    }
}

interface CalleeInterface {
    void doSomething();
}

class Callee implements CalleeInterface {
    void doSomething() {

    }
}
```

แล้วถ้า Contract นี้คนใช้มากกว่า 1 คนละ จะรู้ได้ยังไงว่าใครใช้อะไร
```java
class Caller {
    void process() {
        Callee callee = new Callee();
        callee.doSomething(); 
    }
}

class Caller1 {
    void process() {
        Callee callee = new Callee();
        callee.doSomething2(); 
    }
}

interface CalleeInterface {
    void doSomething();
    void doSomething2();
}

class Callee implements CalleeInterface {
    void doSomething() {

    }
}
```

Caller -- interface --> Callee

ในโลกของ Monolithic มันเร็วเห็นได้เลยว่า break ตรงกัน
แต่ Microservice มันไม่ได้อยู่ด้วยกัน เลยทำให้ใช้เวลากว่าจะรู้

Service1 -- API --> Service 2

ปัญหาส่วนใหญ่ที่เจอสำหรับ Contract
- endpoint ชอบเปลี่ยน
- parameter required เพิ่ม
- parameter เปลี่ยนไป
- เปลี่ยน validate parameter

การจัดการ Contract คือเรื่องของ Compatibility

Contract สำหรับ Provider ไว้สำหรับถ้าเปลี่ยนอะไรแล้วสัญญายังผ่านก็ publish ได้เลย

แล้วใครเป็นคน maintain Contract 
1. Provider เป็นคนดูแลและกำหนด Consumer ต้อง Follow Rule [Provider Contract]
+ เหมาะกับ Centralize service มี (authen & authorize) เช่น openAPI
- ไม่เหมาะกับ Consumer ที่หลากหลาย เช่น PC กับ mobile อยากได้ข้อมูลต่างกัน

2. Consumer เป็นคนกำหนดในมุมผู้ใช้งาน [Consumer Contract]
api เดิม คนใช้งานต่าง ก็ต่าง contract เช่น login แต่อันนึงใช้ อีเมลกับอีกอันใช้ phone number
design contract ตาม ประสบการณ์ใช้งาน (คำนึงถึง UX)
จากเดิมที่เป็นคนกำหนดเปลี่ยนเป็นคนคอย support

3. Consumer-Driven Contract เอา contract ทุกตัวมารวมกันเพื่อให้เห็น contract ทั้งหมด 
ทำให้เราเห็นภาพรวมทั้งหมดจาก A -> B -> C -> D -> E 
ว่าในระบบเชื่อมโยงกันยังไง แต่ละ service มี dependencies กี่เส้น 
ทำให้เรา trace ระหว่าง service ได้


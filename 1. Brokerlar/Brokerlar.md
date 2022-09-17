# 1 Brokerlar

## 1.0 Kirish
Brokerlar biznes g'oyasi (mantig'i) va tashqi dunyo o'rtasidagi dalloldir. Ular ixtiyoriy bir manba yoki tashqi kutubxonaga bog'lanib qolishni oldini olgan holda mahalliy interfeysni qoniqtirib tashqi kutubxonalar, manbalar, xizmatlar va "API"lar o'rab oladi. Interfeyslar biznes talablarini o'zida saqlaydi. 

Brokerlar, umuman olib qaraganda, bir martalik va almashinuvchi qilib yaratilgan - ular texnologiya doimiy rivojlanadi va o'zgaradi degan tushuncha asosida yaratilgan. Shuning uchun, dastur hayotining ma'lum bir nuqtasiga kelib, ular ishni tezroq bajaradigan zamonaviy texnologiyaga o'zgaritiriladi.

Shuningdek, brokerlar sizning biznes dasturingiz maqsadiga tashqi manbaga bog'lanib qolmagan holda ulanuvchanligini ta'minlaydi. 

Misol uchun, siz API loyihasi tuzyapsiz va ushbu loyiha SQL ma'lumotlar bazasidan ma'lumotni o'qiydi va kiritadi. Ma'lum vaqt kelib, siz yaxshiroq, arzonroq variantni tanladingizda, NoSQL dan API loyihangiz uchun ishlatishni boshlamoqchisiz. SQL ga bog'liqlikni ajratib turuvchi broker qavati bo'lishi, NoSQL ga o'tish jarayonini juda oson va tez bo'lishini ta'minlaydi. 

## Xaritada 
Ixtiyoriy dasturda, xoh u mobil, kompyuter, veb-ilova bo'lsin, xoh sodda bir API loyihasi bo'lsin, brokerlar bu dasturning "dumi" joylashadi, chunki ular biz yozgan kod va tashqi dunyo o'rtasidagi aloqaning oxirgi nuqtasidir.

Aytib o'tilgan tashqi dunyo xoh sodda ma'lumotlar ba'zasi bo'lsin, yoki API loyihasi ortida turuvchi mustaqil bir sistema bo'lsin, bularning barchasi Brokerlarning ortida bo'lishi kerak har qanday dasturda. 

Quyida berilgan ma'lum bir APIning sodda arxitekturasida - Brokerlar biznes g'oyasi va tashqi resurslar o'rtasida joylashlashgan. 


<br />
    <p align=center>
        <img src="./Resurslar/Brokers.png" />
    </p>
<br />

## 1.2 Xususiyatlari
Brokerlarning ijrosini boshqaruvchi ba'zi quyidagi qoidalar mavjud: 

### 1.2.0 Mahaliy Interfeysni Qondirishi
Brokerlar mahaliy kontrakt (interfeys)ni qondirishi kerak. Bu ularning implementatsiyasi va ularni ishlatadigan servislarni bog'lanib qolinishi oldini olish uchun kerak. 

Misol uchun, bizda `Student` mahalliy modeli uchun ixtiyoriy CRUD* operatsiyalaridan birini implementatsiyasini talab qiluvchi `IStorageBroker` degan kontrakt (interfeys) bo'lsin. Kontrakt quyidagi ko'rinishda bo'lishi mumkin: 

```csharp
    public partial interface IStorageBroker
    {
        IQueryable<Student> SelectAllStudents();
    }
```

_CRUD (Create, Read, Update, Delete) - ma'lumotlar ba'zasida jadvallar bilan bajariladigan operatsiyalar. Ya'ni, Qo'shish, O'qish, Yangilash va O'chirish operatsiyalarining qisqartmasidir._


Ma'lumotlar bazasi brokerining implementatsiyasi quyidagicha bo'lishi mumkin:


```csharp
    public partial class StorageBroker
    {
        public DbSet<Student> Students { get; set; }

        public IQueryable<Student> SelectAllStudents()
        {
            var broker = new StorageBroker(this.configuration);

            return broker.Students;
        }
    }
```

Mahalliy kontrakt implementatsiyasi Misolda ko'rsatilgani kabi Entity Freymwork ishlatishdan, `Dapper`ga o'xshagan butunlay boshqa texnologiyaga yoki `Oracle` var `PostgreSQL` kabi butunlay boshqa infrastrasturkturaga ixtiyoriy paytda almashtirilishi mumkin.


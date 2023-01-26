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

### 1.2.1 Mantiq Nazorati Mumkin Emas
Brokerlar mantiq nazoratining (if-shart opearatori, while-sikl yoki switch case -tanlash operatori kabi) hech qanday shakliga ega bo'lmasligi kerak ,chunki mantiqni nazorat qilish biznes g'oya hisoblanadi, va u brokerlar qatlamiga emas balki xizmatlar qatlamiga yaxshiroq mos keladi.

Misol uchun, ma'lumotlar bazasidan talabalar ro'yhatini oladigan broker metodi quyidagicha ko'rinishga ega:

```csharp
        public IQueryable<Student> SelectAllStudents() => SelectAll<Student>();
```
EntityFramework-ning `DbSet<T>`-ni chaqiradigan va `Student` kabi modelni qaytaradigan sodda metod. 


### 1.2.2 Xatoliklarga Ishlov Berish Yo'q
Xatoliklarga ishlov berish ham mantiq nazoratining bir shakli. Brokerlar-hech qanday xatoliklarga ishlov bermasligi kerak, aksincha ularning brokerlarga qo'shni bo'lgan xizmatlar qatlamiga o'tishiga imkon berishi kerak, chunki xatoliklarni xizmatlar qatlamida to'g'ri moslashtirish va mahalliylashtirish mumkin.  


### 1.2.3 Sozlamalarga egalik qilish
Brokerlardan oʻz sozlamalarini boshqarish ham talab qilinadi. Ular qaysi tashqi texnologiyani birlashtirgan bo'lishidan qat'iy nazar, sozlama obyektidan sozlamalarni olish va sozlash uchun qaramlilik in'ektsiyasiga ega bo'lishi mumkin.

Misol uchun, ma'lumotlar ombori bilan aloqani muvaffaqiyatli amalga oshirish uchun, ulanishning (string turida) olinishi va ma'lumotlar ombori mijoziga o'tkazilishi talab qilinadi, quyidagicha:

```csharp
    public partial class StorageBroker : EFxceptionsContext, IStorageBroker
    {
        private readonly IConfiguration configuration;

        public StorageBroker(IConfiguration configuration)
        {
            this.configuration = configuration;
            this.Database.Migrate();
        }

        ...
        ...

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
            string connectionString = this.configuration.GetConnectionString("DefaultConnection");
            optionsBuilder.UseSqlServer(connectionString);
        }
    }
```

### 1.2.4 Mahalliy Turlar
Brokerlar  xizmatlar(brokerlarga qo'shni)qatlami tomonidan o'tgan sodda turlar(int,string,char) asosida tashqi model obyektini qurishlari mumkin.

Misol uchun, elektron pochta xabarnomasi brokeridagi  `.Send(...)` funksiyasi  mavzu, content yoki manzil kabi asosiy kiritish parametrlarini talab qilishi quyidagicha bo'ladi.

```csharp
    public async ValueTask SendMailAsync(List<string> recipients, string subject, string content)
    {
        Message message = BuildMessage(recipients, ccRecipients, subject, content);
        await SendEmailMessageAsync(message);
    }
```

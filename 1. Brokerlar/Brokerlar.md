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
Brokerlar  xizmatlar (brokerlarga qo'shni) qatlami tomonidan o'tgan sodda turlar (int,string,char) asosida tashqi model obyektini qurishlari mumkin.

Misol uchun, elektron pochta xabarnomasi brokeridagi  `.Send(...)` funksiyasi  mavzu, content yoki manzil kabi asosiy kiritish parametrlarini talab qilishi quyidagicha bo'ladi.

```csharp
    public async ValueTask SendMailAsync(List<string> recipients, string subject, string content)
    {
        Message message = BuildMessage(recipients, ccRecipients, subject, content);
        await SendEmailMessageAsync(message);
    }
```

Sodda turdagi (int,string,char) kirish parametrlari xizmatlar(brokerga qo'shni qatlam) va tashqi modellar orasidagi kuchli bog'liqlikni kamaytiradi.
Hattoki, broker sizning dasturingiz va tashqi RESTful API ni bog'lash nuqtasi bo'lib xizmat qilsada, mahalliy modellarni API ga jo'natiladigan yoki qabul qilinadigan modellar bilan bir xil qilish maslahat beriladi. Bu jarayonda NuGet kutubxonalari, dastur ichida yaratilgan kutubxonalar yoki umumiy loyiha turlaridan foydalanish tavsiya etilmaydi.

### 1.2.5 Nomlashdagi Qoidalar
Brokerlar uchun tuzilgan shartnomalar, uning barcha vazifalarini ko'rsatishi uchun imkon qadar generic (umumiy) bo'lishi kerak; masalan, ma'lum bir texnologiya yoki infratuzilmani ko'rsatish uchun `ISqlStorageBroker` ning o'rniga `IStorageBroker` dan fordalanamiz .

Yagona ombor brokeri orqali shartnoma nomi bilan bir xil nomni saqlab qolish qulayroq.Ammo brokerlarni implementatsiya (dasturga kiritganimizda) qilganimizda, bularning barchasi shunga o'xshash vazifalarni bajaradigan brokerlarning miqdoriga bo'g'liq bo'ladi.Bizning holatda, bizda "IStorageBroker" interfeysi mavjud, shuning uchun uning nomi "StorageBroker" bo'ladi.

Agarda, ilovangiz bir nechta queues(navbat), storage(ombor) yoki elektron pochta xizmati provayderlarini qo'llab-quvvatlasa, siz brokerning umumiy maqsadini belgilashni boshlashingiz kerak; masalan, `IQueueBroker`ning `OrdersQueueBroker` va `NotificationQueueBroker` kabi implementatsiya (sozlama) lari bo'ladi.

Agar implementatsiya (sozlama) bir xil model va biznes g'oyasiga qaratilgan bo'lsa, texnologiyaga o'tish ko'proq mos kelishi mumkin.Masalan, `IStorageBroker`, ikki xil aniq `SqlStorageBroker` va `MongoStorageBroker` kabi  dasturlar bilan  shartnoma tuzishi mumkin. Bu esa ishlab chiqarish infratuzilmasi xarajatlarini kamaytirish niyatida bo'lgan holatlarga uchun  xosdir.

### 1.2.6 Til
Brokerlar o'zlari qo'llab-quvvatlaydigan tashqi texnologiyalar tilida gaplashadilar.
Masalan, SQL dagi(ma'lumotlar ombori) `Select` so'roviga mos kelish uchun  ombor brokerida biz `SelectById` dan foydalanamiz, queue (navbat) brokerida esa tilga mos kelish uchun `Enqueue` dan.

Agar broker API so'nggi nuqtasida foydalanilsa, u `POST`, `GET` yoki `PUT` kabi RESTFul semantikasiga amal qilishi kerak. Quyidagi misoldagidek:

```csharp

    public async ValueTask<Student> PostStudentAsync(Student student) =>
        await this.PostAsync(RelativeUrl, student);

```
### 1.2.7 Atrofdagilar
Brokerlar boshqa brokerlarni chaqira olmaydi; brokerlar mavhumlikning birinchi nuqtasi bo'lgani uchun ular sozlamalarga kirish modelidan boshqa hech qanday qo'shimcha mavhumlik yoki bog'liqlikni talab qilmaydi.

Brokerlar xizmatlar qatlamiga ham bog'liq bo'lishi mumkin emas, chunki har qanday tizimdagi mantiq xizmatlar qatlamidan brokerlarga keladi, aksincha bo'lishi mumkin emas.

Misol uchun, mikroservis queue(navbat)ga a'zo bo'lishi kerak bo'lsa ham, brokerlar kiruvchi hodisalarni qayta ishlash uchun Listener(tinglovchi) usulini oldinga o'tkazadilar, lekin ishlov berish mantiqini ta'minlaydigan xizmatlarni chaqirmaydilar.

Bu erda umumiy qoida shundan iboratki, faqat xizmatlar qatlami brokerlarni chaqirishi mumkin; brokerlar faqat tashqi mahalliy bog'liqliklarni chaqirishlari mumkin.

## 1.3 Qismlar

Ombor brokerlari kabi bir nechta sub'ektlarni qo'llab-quvvatlovchi brokerlar har bir entity(model) uchun mas'uliyatni taqsimlash uchun qismlarga bo'linishi kerak.

Misol uchun, agar bizda "Student" va "Teacher" entity(modellar)i uchun barcha CRUD(Create,Read,Update,Delete) amallarini ta'minlaydigan ombor brokerimiz bo'lsa, u holda fayllarni tashkil qilish quyidagicha bo'lishi kerak:

- IStorageBroker.cs
  - IStorageBroker.Students.cs
  - IStorageBroker.Teachers.cs
- StorageBroker.cs
  - StorageBroker.Students.cs
  - StorageBroker.Teachers.cs


Sinfni qismlarga ajratish - maxsus broker har bir obyektga  alohida e'tibor qaratadi, bu esa dasturiy ta'minotning barqarorligini ancha yuqori qiladi.

Ammo broker fayllari va papkalarni nomlash qoidalari qat'iy ravishda ular qo'llab-quvvatlaydigan ob'ektlarning ko'pligiga va qo'llab-quvvatlanadigan umumiy resursning o'ziga xosligiga qaratilgan.

Masalan, biz `IStorageBroker.Students.cs` deymiz. Shuningdek, biz `IEmailBroker` yoki `IQueueBroker.Notifications.cs` ham deb atashimiz mumkin.

Xuddi shu qoidalar ushbu brokerlarni o'z ichiga olgan folders(papkalar) yoki namespaceslarga nisbatan qo'llaniladi.

Masalan, quyidagicha:

```csharp
namespace OtripleS.Web.Api.Brokers.Storages
{
    ...
}
```

Va bunday:

```csharp
namespace OtripleS.Web.Api.Brokers.Queues
{
    ...
}
```

### 1.4.0 Entity Brokerlar
Entity brokerlari dasturning biznes talablarini bajarish uchun zarur bo'lgan tashqi resurslar bilan integratsiya(a'loqa) nuqtalarini ta'minlaydi.

Misol uchun, entity brokerlar storage(ombor) bilan integratsiya(a'loqa) qilish orqali, ma'lumotlar ombori(database)dan ma'lumotlarni olish va u yerga saqlashni ta'minlaydi. 

Shuningdek, Entity brokerlari queue(navbat) brokerlariga o'xshab, o'zlarining biznes mantig'ini bajarish uchun boshqa xizmatlar foydalanish va qayta ishlash uchun xabarlarni navbatga yuborish uchun integratsiya ham nuqtasini ta'minlaydi.

Broker bilan qo'shni service(xizmat) qavati faqat entity brokerlarini chaqirishi  mumkin, chunki ular logikani davom etishdan oldin qabul qilgan yoki taqdim etgan ma'lumotlarni bir neshta tekshirishdan o'takzishni talab qiladi.

### 1.4.1 Yordamchi Brokerlar
Yordamchi brokerlar umumiy maqsadli brokerlar bo'lib, ular yordamchi xizmatlar uchun funksionallikni ta'minlaydi, ammo ularni boshqa tizimlardan ajratib turadigan hech qanday xususiyatga ega emas.

`DateTimeBroker` yordamchi brokerlariga yaxshi misol boʻlishi mumkin – bu biznes qatlamining tizim sanasi vaqtiga kuchli bogʻliqligini yoʻqotish uchun maxsus yaratilgan broker.

Vaqt brokerlari hech qanday modelni maqsad qilib qo'ymaydi va ular ko'plab tizimlarda deyarli bir xil.

Yordamchi brokerlarining yana bir misoli `LoggingBroker`.Ular tizim muhandislariga tizim bo'ylab ma'lumotlarning umumiy oqimini tasavvur qilishlari va har qanday muammo yuzaga kelganda xabardor bo'lishlari uchun har qanday fayllarga ma'lumotlarni yozib boradi .

Entity Brokerlardan farqli o'laroq, yordamchi brokerlar biznes qatlamining har qanday qavatida  chaqirilishi mumkin:  foundation(asos), processing(ishlov berish), orchestration(orkestratsiya), coordination(muvofiqlashtirish), management(boshqaruv) yoki aggregation(yig'ish) service(xizmat)larida. Buning sababi, logging brokerlari xizmatlar qatlamlariga xatolarni qayd qilish yoki sanani va boshqa qo'llab-quvvatlovchi funksiyalarni hisoblash uchun zarur bo'lgan barcha imkoniyatlarni taqdim etish uchun tizimda yordamchi komponent sifatida talab qilinadi.

Brokerlarning haqiqiy misollarini [OtripleS](https://github.com/hassanhabib/OtripleS/tree/master/OtripleS.Web.Api/Brokers) loyihasida ko'rishingiz mumkin.

## 1.5 Ishlatilishi

Bu yerda `Student` ob’ektiga tegishli barcha CRUD(Create,Read,Update,Delete) amallari uchun ombor brokerining real hayotda tatbiq etilishi:

###### IStorageBroker.cs:
```csharp
namespace OtripleS.Web.Api.Brokers.Storage
{
    public partial interface IStorageBroker
    {
    }
}

```

###### StorageBroker.cs:
```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using EFxceptions.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using OtripleS.Web.Api.Models.Users;

namespace OtripleS.Web.Api.Brokers.Storages
{
    public partial class StorageBroker : EFxceptionsIdentityContext<User, Role, Guid>, IStorageBroker
    {
        private readonly IConfiguration configuration;

        public StorageBroker(IConfiguration configuration)
        {
            this.configuration = configuration;
            this.Database.Migrate();
        }

        private async ValueTask<T> InsertAsync<T>(T @object)
        {
            this.Entry(@object).State = EntityState.Added;
            await this.SaveChangesAsync();

            return @object;
        }

        private IQueryable<T> SelectAll<T>() where T : class => this.Set<T>();

        private async ValueTask<T> SelectAsync<T>(params object[] @objectIds) where T : class =>
            await this.FindAsync<T>(objectIds);

        private async ValueTask<T> UpdateAsync<T>(T @object)
        {
            this.Entry(@object).State = EntityState.Modified;
            await this.SaveChangesAsync();

            return @object;
        }

        private async ValueTask<T> DeleteAsync<T>(T @object)
        {
            this.Entry(@object).State = EntityState.Deleted;
            await this.SaveChangesAsync();
     
            return @object;
        }

        ...

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
            string connectionString = this.configuration.GetConnectionString("DefaultConnection");
            optionsBuilder.UseSqlServer(connectionString);
        }
    }
}
```

###### IStorageBroker.Students.cs:
```csharp
using system;
using system.Linq;
using system.Threading.Tasks;
using OtripleS.Web.Api.Models.Students;

namespace OtripleS.Web.Api.Brokers.Storage
{
    public partial interface IStorageBroker
    {
        public ValueTask<Student> InsertStudentAsync(Student student);
        public IQueryable<Student> SelectAllStudents();
        public ValueTask<Student> SelectStudentByIdAsync(Guid studentId);
        public ValueTask<Student> UpdateStudentAsync(Student student);
        public ValueTask<Student> DeleteStudentAsync(Student student);
    }
}
``` 

###### StorageBroker.Students.cs:
```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using OtripleS.Web.Api.Models.Students;

namespace OtripleS.Web.Api.Brokers.Storages
{
    public partial class StorageBroker
    {
        public DbSet<Student> Students { get; set; }

        public async ValueTask<Student> InsertStudentAsync(Student student) =>
            await InsertAsync(student);

        public IQueryable<Student> SelectAllStudents() => SelectAll<Student>();

        public async ValueTask<Student> SelectStudentByIdAsync(Guid studentId) =>
            await SelectAsync<Student>(studentId);

        public async ValueTask<Student> UpdateStudentAsync(Student student) =>
            await UpdateAsync(student);

        public async ValueTask<Student> DeleteStudentAsync(Student student) =>
            await DeleteAsync(student);
    }
}
```

## 1.6 Xulosa
Brokerlar sizning biznes g'oyangiz va tashqi dunyo o'rtasidagi mavhumlikning dastlabki qatlamidir. Lekin ular mavhumlikning yagona qatlami emas; chunki sizning brokerlaringiz orqali ularga qo'shni service(xizmatlar)ingizga o'ta oladigan bir nechta mahalliy modellar bo'ladi. Biznes g'oya  doirasidan tashqarida har qanday mapping(ko'chirib o'tkazish)dan qochish tabiiydir; bizning holatda esa bu foundation service(xizmatlar)idir. 

Masalan, saqlash brokerida (ORM dan qat'iy nazar, masalan, EntityFramework) ba'zi mahalliy xatoliklar paydo bo'ladi, masalan, "DbUpdateException" yoki "SqlException". Bunday holda, ushbu xatoliklar va ularni mahalliy xatolik modellariga aylantirish uchun bizning asosiy mantiqimiz o'rtasida xaritachi rolini o'ynash uchun bizga yana bir abstraksiya qatlami kerak.

Bu mas'uliyatga brokerga qo'shni service(xizmat)lar javobgar. Men ularni foundation(asos) service(xizmat)lari deb ham atayman, bu xizmatlar mahalliy modellar va shartnomalardan iborat bo'lgan asosiy mantiqingiz oldidagi so'nggi mavhumlik nuqtasidir.

## 1.7 Tez-tez so'raladigan savollar
Vaqt o'tishi bilan men faoliyatim davomida ishlagan muhandislardan ba'zi umumiy savollar paydo bo'ldi. Ushbu savollarning ba'zilari bir necha marta takrorlanganligi sababli, men brokerlarning boshqa istiqbollari haqida hamma bilishi uchun ularni shu yerda ttaqdim etishni foydali bo'ladi deb o'yladim.

#### 1.7.0 Brokerlar Patterni Repository Patterni bilan bir xilmi?
Bunday emas, hech bo'lmaganda jarayonlar nuqtai nazardan, brokerlar repositoridan ko'ra generic(umumiy)roq.

Repositariylar odatda saqlashga o'xshash operatsiyalarni, asosan ma'lumotlar bazalariga qaratilgan. Ammo brokerlar elektron pochta xizmatlari, queue(navbatlar) va boshqa APIlar kabi har qanday tashqi qaramlik bilan integratsiya nuqtasi bo'lishi mumkin.

Brokerlarga o'xshash dizayn pattern bu Unit of Work dizayn patterni.U, asosan, nomni biron bir jarayon bilan bog'lamasdan, umumiy jarayonga e'tibor qaratadi.

Bu dizayn patterlarning barchasi, bir xil SOLID tamoyillarini amalga oshirishga harakat qiladi: tashvishlarni ajratish, qaramlikni in'ektsiya qilish va yagona javobgarlik.

Ammo SOLID aniq ko'rsatmalar emas, balki printsiplar bo'lganligi sababli, ushbu printsipga erishish uchun barcha turli xil implementatsiyalar va patternlarni nazarda tutiladi.
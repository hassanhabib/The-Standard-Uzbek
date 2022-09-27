# 2.1 Asos Xizmatlari (Brokerga O'zaro Bog'langan)

## 2.1.0 Kirish

Asos xizmatlari bu sizning biznesingizning g'oyasi(biznes logika) va brokerlar o'rtasidagi birinchi bog'lanish nuqtasi hisoblanadi.

Umimiy qilib aytganda, yuqori darajadagi biznes g'oyasi(biznes logika) qayerda sodir bo'lsa, shunday holatda biznes g'oyasi(biznes logika) va mavhumlik qatlami jarayonlarni qayta ishlash uchun birlashishi, brokerga o'zaro bog'liq xizmatlari bo'ladi. Keyingi bo'limda qayta ishlovchi xizmatlarni yaratishni boshlaganimizda, yuqori darajadagi biznes g'oyasi(biznes logika) haqaida gaplashamiz.

Brokerga o'zaro bog'liq xizmatlarining asosiy vazifasi, tizim orqali kiruvchi va chiquvchi ma'lumotlarning tizimli, mantiqiy va tashqi tomondan tasdiqlanganligini va tekshirilganligini ta'minlashdir.

Shuningdek, brokerga o'zaro bog'liq xizmatlarni broker taqdim etadigan sodda amallar ustida boshlang'ich tekshiruvni amalga oshiruvchi xizmatlar deb qarashingiz ham mumkin.

Misol uchun, agar saqlash brokeri `InsertStudentAsync(Student student)` ni metod sifatida taklif qilsa, keyin brokerga o'zaro bog'liq xizmatlar quyidagi kabi bo'lishi mumkin:

```csharp
public async ValueTask<Student> AddStudentAsync(Student student)
{
	ValidateStudent(student);

	return await this.storageBroker.InsertStudentAsync(student);
}
```

Bu brokerga o'zaro bog'liq xizmatlar brokerlar taklif qilayotgan mavjud sodda jarayonlar ustida qo'shimcha tekshirish qatlamidan boshqa narsa qilmaydi.

## 2.1.1 Xaritada

Brokerga o'zaro bog'liq xizmatlar sizning brokerlaringiz va ilovangizni qolgan qismning o'rtasida joylashgan bo'lib va chap tomonda yuqori darajadagi biznes g'oyasi(biznes logika) qayta ishlash xizmatlari, orkestratsiya, muvofiqlashtirish, yig'ish yoki boshqarish xizmatlari, oddiy boshqaruvchi, UI tarkibiy qismi yoki boshqa ma'lumotlarga ta'sir qilish texnologiyasi bo'lishi mumkin.

<br/>
	<p align=center>
		<img src="https://user-images.githubusercontent.com/1453985/100716772-00eec800-336e-11eb-9064-8bfe2f8e3be2.png" />
	</p>
<br/>

## 2.1.2 Xususiyatlari

Umuman olganda, asos yoki brokerga o'zaro bo'gliq xizmatlarni rivojlantirish va integratsiyasini boshqaruvchi aniq xususiyatlari mavjud.

Boshqalarga nisbatan asos xizmatlari ko'proq e'tiborni tekshirishga qaratadi, chunki bu ularning asosiy vazifazi hisoblanadi. Ya'ni, asos xizmatlari tizim orqali kiruvchi va chiquvchi barcha ma'lumotlarni hech qanday muammosiz xavfsiz qayta ishlash uchun yaxshi holatda bo'lishini ta'minlaydi. 

Quyida brokerga o'zaro bog'liq xizmatlarni boshqaradigan xususiyatlar va qoidalar:

### 2.1.2.0 Juda Oddiy

Yuqori darajali biznes g'oyaning(biznes logika) erishish uchun brokerga o'zaro bog'liq xizmatlar bir nechta oddiy jarayonlarni aralashtirib yuborishiga yo'l qo'yilmaydi.

Misol uchun, brokerga o'zaro bog'liq xizmatlar _upsert_ funksiyasini taklif qila olmaydi, bu esa obʼyekt mavjudligini va har qanday xotirada yangilanganligini taʼminlash uchun “Tanlash” jarayonlarini “Yangilash” yoki “Qoʻshish” jarayonlari bilan natijaga ko'ra birlashtira olmaydi.

Lekin, ular bog'liqlikni chaqiruvchi atrofida tekshirish va istisnolar bilan ishlash (va o'zgartirish) imkoniyatini taklif qiladi, misol tariqasida:

```csharp
public ValueTask<Student> AddStudentAsync(Student student) =>
TryCatch(async () =>
{
	ValidateStudent(student);

	return await this.storageBroker.InsertStudentAsync(student);
});
```

Yuqoridagi funksiyada ko'rishingiz mumkin, `ValidateStudent` funksiyasini chaqirishdan oldin `TryCatch` bloki keladi.
"TryCatch" bloki bu men Istisno Shovqinni Bekor Qilish deb atagan namunadir. Bu bo'lim haqida tez fursatda muhokama qilamiz.

Ammo tekshirish funksiyasi kiruvchi ma'lumotlardagi har bir xususiyatni oddiy broker jarayoniga o'tkazishdan oldin tekshirilishini ta'minlaydi, ya'ni aynan shu holatda `InsertStudentAsync`.
# 2.1 Asos Xizmatlari (Brokerga o'zaro bog'langan)

## 2.1.0 Kirish

Asos xizmatlari bu sizning ishingizning mantig'i(business logic) va brokerlar o'rtasidagi birinchi bog'lanish nuqtasi hisoblanadi.

Umimiy qilib aytganda, yuqori darajadagi ish mantig'i(business logic) qayerda sodir bo'lsa, usha holatda ish mantig'i(business logic) va mavhumlik qatlami jarayonlarni qayta ishlash uchun birlashishi, brokerga o'zaro bog'liq xizmatlari bo'ladi. Keyingi bo'limda qayta ishlovchi xizmatlarni yaratishni boshlaganimizda, yuqori darajadagi ish mantig'i(business logic) haqaida gaplashamiz.

Brokerga o'zaro bog'liq xizmatlarining asosiy vazifasi, tizim orqali kiruvchi va chiquvchi ma'lumotlarning tizimli, mantiqiy va tashqi tomondan tasdiqlanganligini va tekshirilganligini ta'minlashdir.

Shuningdek, brokerga o'zaro bog'liq xizmatlar sodda jarayonlar ustida tasdiqlash qatlami sifatida brokerlar allaqachon taklif qilgananini o'ylashingiz mumkin.

Misol uchun, agar saqlash brokeri `InsertStudentAsync(Student student)` ni uslub sifatida taklif qilsa, keyin brokerga o'zaro bog'liq xizmatlar biror nima taklif qiladi:

```csharp
public async ValueTask<Student> AddStudentAsync(Student student)
{
	ValidateStudent(student);

	return await this.storageBroker.InsertStudentAsync(student);
}
```

Bu brokerga o'zaro bog'liq xizmatlar brokerlar taklif qilayotgan mavjud sodda jarayonlar ustida qo'shimcha tasdiqlash qatlamidan boshqa narsa qilmaydi.

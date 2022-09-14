## 2.2 Qayta ishlash xizmatlari (Yuqori tartibli biznes mantig'i)

## 2.2.0 Kirish

Qayta ishlash xizmatlari yuqori darajadagi biznes mantig'i amalga oshiriladigan qatlamdir. Ular yangi funksionallikni joriy qilish uchun oʻzlarining asos xizmatlaridan oddiy darajadagi funksiyani birlashtirishlari mumkin. Ular, shuningdek, bitta oddiy funktsiyani chaqirishlari va biroz qo'shilgan biznes mantig'i bilan natijani o'zgartirishlari mumkin. Va ba'zida ishlov berish xizmatlari umumiy arxitekturaga muvozanatni joriy qilish uchun o'tish vositasi sifatida mavjud.

Qayta ishlash xizmatlari biznes ehtiyojlaringizga qarab ixtiyoriydir - oddiy CRUD operatsiyali APIda ishlov berish xizmatlari va undan keyingi barcha xizmatlar toifalari o'z faoliyatini to'xtatadi, chunki bu vaqtda biznes mantig'ining yuqori tartibiga ehtiyoj qolmaydi.

Qayta ishlash xizmati funksiyasi qanday ko'rinishda bo'lishiga misol:

```csharp
public ValueTask<Student> UpsertStudentAsync(Student student) =>
TryCatch(async () =>
{
    ValidateStudent(student);

    IQueryable<Student> allStudents =
        this.studentService.RetrieveAllStudents();

    bool studentExists = allStudents.Any(retrievedStudent =>
        retrievedStudent.Id == student.Id);

    return studentExists switch {
        false => await this.studentService.RegisterStudentAsync(student),
        _ => await this.studentService.ModifyStudentAsync(student.Id)
    };
});
```

Qayta ishlash xizmatlari Foundation xizmatlarini mavjud sodda operatsiyalar ustiga tekshirish qatlamidan boshqa narsa qilmaydi. Bu shuni anglatadiki, ishlov berish xizmatlarining funktsiyalari oddiy emas va ular faqat mahalliy modellar bilan shug'ullanadi, bu haqida keyingi bo'limlarda muhokama qilamiz.
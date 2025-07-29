```python
import base64
import os
from mistralai import Mistral

def encode_pdf(pdf_path):
    """
    Encode the PDF at `pdf_path` into a Base64 string.
    Returns None on error.
    """
    try:
        # افتح الملف بوضعية القراءة الثنائية "rb" لقراءة البيانات الخام كبايتات
        with open(pdf_path, "rb") as pdf_file:
            # اقرأ كل البايتات ثم حولها إلى سلسلة Base64
            encoded_bytes = base64.b64encode(pdf_file.read())
            # حوّل النتيجة من بايتس إلى نص utf-8
            return encoded_bytes.decode("utf-8")
    except FileNotFoundError:
        # إذا لم يجد الملف، يطبع رسالة خطأ ويرجع None
        print(f"Error: The file {pdf_path} was not found.")
        return None
    except Exception as e:
        # أي استثناء آخر يُطبع وتُعاد None
        print(f"Error: {e}")
        return None

# المسار إلى ملف البي دي إف
pdf_path = "document-1-5.pdf"

# استدعاء دالة الترميز
base64_pdf = encode_pdf(pdf_path)

# تأكد من أن الترميز نَجَح قبل المتابعة
if base64_pdf:
    # أدخل مفتاح الـ API مباشرة هنا (بدون .env)
    api_key = "SB6B01uJMUWzpwu7MsqStV3UZaoj0zOB"

    # أنشئ عميل Mistral باستخدام المفتاح
    client = Mistral(api_key=api_key)

    # استدعاء خدمة الـ OCR لمعالجة المستند
    ocr_response = client.ocr.process(
        model="mistral-ocr-latest",
        document={
            "type": "document_url",
            # أرسل مستند PDF كسلسلة Base64 عبر data URI
            "document_url": f"data:application/pdf;base64,{base64_pdf}"
        },
        include_image_base64=False
    )

    # تحديد اسم ملف الماركداون للإخراج
    output_filename = "output.md"

    try:
        # افتح ملف الماركداون للكتابة مع ترميز UTF-8 لدعم العربية
        with open(output_filename, "w", encoding="utf-8") as md_file:
            # لكل صفحة في استجابة الـ OCR...
            for page in ocr_response.pages:
                page_number = page.index + 1  # رقمنة الصفحات تبدأ من 0
                content = page.markdown     # محتوى الماركداون المولد
                # اكتب عنوان الصفحة ثم المحتوى
                md_file.write(f"## Page {page_number}\n\n")
                md_file.write(content + "\n\n")
        print(f"تم حفظ النص بنجاح في الملف: {output_filename}")
    except Exception as e:
        print(f"حدث خطأ أثناء كتابة الملف: {e}")
```

---

### الأمور الضرورية لتشغيل السكريبت بنجاح

* تثبيت Python 3.7 أو أحدث والتأكد من أن إصدار Python صحيح:

  ```bash
  python --version
  ```

  ([Python documentation][1])

* تثبيت مكتبة Mistral الرسمية:

  ```bash
  pip install mistralai
  ```

  ([Mistral AI][2])

* لا تستخدم ملفات `.env` أو بيئات تشغيل منفصلة؛ ضع `api_key` مباشرةً في الكود أو مرره كمتغير عند التشغيل.

* التأكد من توفر ملف `document-1-5.pdf` في نفس مجلد السكريبت أو تعديل المسار وفقًا لذلك؛ فتح الملف في وضعية `"rb"` ضروري ([GeeksforGeeks][3]).

* استيراد وحدة `base64` الافتراضية من بايثون لتحويل البايتات إلى سلسلة Base64:
  هذه الوحدة مدعومة في بايثون بدون تنصيب إضافي ([Python documentation][4]).

* التعامل مع استثناء `FileNotFoundError` عند قراءة الملف لتجنب انهيار البرنامج:
  `FileNotFoundError` هو استثناء مدمج في بايثون يُرمى عند عدم وجود الملف ([Python documentation][5]).

* تحديد ترميز UTF-8 عند كتابة ملف Markdown لضمان دعم النصوص العربية:
  تمرير `encoding="utf-8"` مع `open()` هو أفضل ممارسة لتوافق المنصات المختلفة ([Python documentation][6]).

* التأكد من اتصال إنترنت مستقر لإجراء طلبات API إلى خوادم Mistral.

* التحقق من صلاحية المفتاح `api_key` في حساب Mistral الخاص بك وأنه لم تنتهِ صلاحيته.

* مراجعة مستندات Mistral OCR للتأكد من اسم النموذج `"mistral-ocr-latest"` وأي تغييرات محتملة في الواجهة البرمجية ([Mistral AI][7], [cohorte.co][8]).

---

### شرح مبسط (لغير المختصين)

1. **تشفير الملف**: نفتح ملف PDF كبيانات ونحوّله إلى نص طويل مشوّب (Base64) ليمكن إرساله عبر الإنترنت.
2. **توصيل الخدمة**: نسجل دخولنا في خدمة "Mistrال" بإعطاء السكربت مفتاح سري.
3. **طلب التحويل**: نرسل النص المشوّب إلى الخادم ليحلل صفحات PDF ويستخرج النص.
4. **حفظ النتيجة**: نكتب كل صفحة كنص في ملف Markdown منسّق، جاهز للقراءة أو المعالجة لاحقًا.

[1]: https://docs.python.org/3/whatsnew/3.7.html?utm_source=chatgpt.com "What's New In Python 3.7 — Python 3.13.4 documentation"
[2]: https://docs.mistral.ai/getting-started/clients/?utm_source=chatgpt.com "Clients | Mistral AI"
[3]: https://www.geeksforgeeks.org/python/reading-binary-files-in-python/?utm_source=chatgpt.com "Reading binary files in Python - GeeksforGeeks"
[4]: https://docs.python.org/3/library/base64.html?utm_source=chatgpt.com "base64 — Base16, Base32, Base64, Base85 Data Encodings ..."
[5]: https://docs.python.org/3/library/exceptions.html?utm_source=chatgpt.com "Built-in Exceptions — Python 3.13.5 documentation"
[6]: https://docs.python.org/3/library/io.html?utm_source=chatgpt.com "io — Core tools for working with streams ... - Python documentation"
[7]: https://docs.mistral.ai/capabilities/OCR/basic_ocr/?utm_source=chatgpt.com "Basic OCR - Mistral AI"
[8]: https://www.cohorte.co/blog/a-step-by-step-guide-to-using-mistral-ocr?utm_source=chatgpt.com "A Step-by-Step Guide to Using Mistral OCR - Cohorte Projects"


## نسخة أبسط من الكود:
```python

import base64
from mistralai import Mistral

def encode_pdf(pdf_path):
    """Encode the PDF at pdf_path into a Base64 string."""
    with open(pdf_path, "rb") as f:
        return base64.b64encode(f.read()).decode()

# مسار ملف PDF
pdf_path = "document-1-5.pdf"
base64_pdf = encode_pdf(pdf_path)

# مفتاح API وإنشاء عميل Mistral
api_key = "SB6B01uJMUWzpwu7MsqStV3UZaoj0zOB"
client = Mistral(api_key=api_key)

# إرسال المستند إلى خدمة OCR
ocr_response = client.ocr.process(
    model="mistral-ocr-latest",
    document={
        "type": "document_url",
        "document_url": f"data:application/pdf;base64,{base64_pdf}"
    },
    include_image_base64=False
)

# كتابة النتائج في ملف Markdown
with open("output.md", "w", encoding="utf-8") as md_file:
    for page in ocr_response.pages:
        md_file.write(f"## Page {page.index + 1}\n\n")
        md_file.write(page.markdown + "\n\n")

print("تم حفظ النص في output.md")
```

**ملاحظات بسيطة:**
- لا يوجد هنا معالجة للأخطاء أو تحقق من القيم، لذا تأكد من وجود الملف والمفتاح الصحيح قبل التشغيل.
- يمكنك تعديل `"document-1-5.pdf"` واسم الملف الناتج `"output.md"` حسب الحاجة.
::contentReference[oaicite:0]{index=0}

# مِعيار — تحليل البيانات القانونية للقطاع غير الربحي السعودي

> المؤشرات والتصنيفات مخرجات تحليلية للمراجعة، وليست استشارة قانونية أو حكمًا على امتثال أي جهة.

## ترتيب الملفات

```text
.
├── README.md                 # هذا الملف
├── ai_usage_log.md           # توثيق استخدام الذكاء الاصطناعي
├── analysis.ipynb            # دفتر التحليل (يشغّل run_pipeline.py ثم يعرض النتائج)
├── notebooks/
│   ├── 01_clean_legal_articles.ipynb    # تنظيف ملف المواد (Google Colab)
│   └── 02_model_matching.ipynb          # نموذج المطابقة والتجميع (Google Colab + GPU)
├── data_raw/                 # مدخلات دفتر التنظيف (7 ملفات)
├── data_clean/
│   └── legal_articles_full.csv          # ناتج التنظيف = مدخل النموذج
├── outputs/                  # الجداول والنتائج
│   └── figures/              # الرسوم (تُنشأ بـ scripts/make_figures.py)
├── report/
│   └── executive_summary.pdf
├── scripts/make_figures.py   # يرسم الرسومات الخمس في outputs/figures
├── src/, tests/, run_pipeline.py, requirements.txt   # كود التحليل واختباراته
└── index.html, netlify.toml, SUPABASE_SETUP.sql       # واجهة الموقع
```

## التشغيل

### 1) التنظيف — `notebooks/01_clean_legal_articles.ipynb`
في Google Colab: Runtime ← Run all، ثم ارفعي **الملفات السبعة في `data_raw/`** معًا (تحديد متعدد):
- `legal_articles_full_updated.csv` — ملف المواد من مشروع الفريق (47 مادة من قواعد الحوكمة و129 من اللائحة التنفيذية).
- `legal_articles_errors_additions.xlsx` — الأسطر الدخيلة، و«ء» الزائدة، والأسطر الساقطة من القراءة الآلية.
- `مراجعة_المواضيع.xlsx` — المواضيع «الواضحة» المعتمدة.
- `مراجعة_القانوني_المواضيع.xlsx` و`مفتاح_مراجعة_القانوني_لا_يُرسل.csv` — قرارات المختص القانوني ومفتاحها.
- `legal topic classification.py` و`legal_text_extraction.py` — كود تصنيف المواضيع من مشروع الفريق.

**الناتج:** ينزل `legal_articles_full.csv` (المحفوظ في `data_clean/`).

### 2) النموذج — `notebooks/02_model_matching.ipynb`
Runtime ← Change runtime type ← GPU، ثم Run all، وارفعي `data_clean/legal_articles_full.csv` (المدخل الوحيد).
- المطابقة بـ BAAI/bge-m3 (الطريقة المعتمدة) مع حد التشابه 0.75، وخط الأساس TF-IDF، والتجميع الموضوعي، وأداة العرض في آخر خلية.
- نتائج تقييم النموذج بالمراجعة البشرية مكتوبة في أول الدفتر، وتفاصيلها في `outputs/نتائج_النموذج.xlsx`.
- مخرجات النموذج في `outputs/`: `نتائج_النموذج.xlsx`، `المطابقات_النهائية_bge_m3.csv`، `clusters_bge_m3.csv`، `clusters_vs_topics.xlsx`.

### 3) التحليل — `analysis.ipynb` و`run_pipeline.py` و`src/`
```bash
python -m pip install -r requirements.txt
python run_pipeline.py
python -m pytest tests/
```
يحتاج Python 3.12 أو أحدث، وTesseract OCR بالعربية لاستخراج النصوص من PDF (`sudo apt-get install -y tesseract-ocr tesseract-ocr-ara`). الملفات الأصلية توضع في المسارات المحددة في `src/config.py`.

### 4) الرسومات — `scripts/make_figures.py`
```bash
python scripts/make_figures.py
```
يرسم في `outputs/figures/` رسوم `outputs/statistical_report_dashboard.xlsx` الخمس: رسوم الكيانات من أوراق ملف Excel، ورسما المواضيع من `data_clean/legal_articles_full.csv` (176 مادة، 25 موضوعًا).

# الأنماط والأعلام

التعبيرات العادية هي أنماط توفر طريقة فعالة للبحث والاستبدال في النص.

في ال JavaScript, أنها تكون متوفرة عبر [RegExp](mdn:js/RegExp), وكذلك يمكن دمجها في طرق النصوص.

## التعبيرات المنتظمة

النعبير المنتظم (ويمكن أختصاره الي "regexp", او "reg") يتكون من _نمط_ و*أعلام* أختيارية.

هناك نوعان من بناء الجملة يمكن استخدامهما لإنشاء كائن تعبير منتظم.

طريقة الكتابة "الطويلة":

```js
regexp = new RegExp('نمط', 'أعلام');
```

وطريقة الكتابة "القصيرة", بأستخدام "/":

```js
regexp = /نمط/; // بدون أعلام
regexp = /نمط/gim; // أستخدام الاعلام g,m والعلم i (سيتم تغطيته فيما بعد)
```

هذا الرمز "/.../" يعرف ال JavaScript أننا سننشئ تعبير منتظم. وهي كذلك مثل علامات التنصيص في النصوص.

في الحالتين يصبح ال `regexp` نموذج مبني من ال `RegExp`.

الأختلاف الرئيسي بين الطريقتين أن النمط بأستخدام ال `/.../` لا يسمح لك بكتابة تعبيرات بداخله (مثل التعبير `${...}`). في هذه الحاله تكون الجمله ثابتة.

Slashes are used when we know the regular expression at the code writing time -- and that's the most common situation. While `new RegExp` is more often used when we need to create a regexp "on the fly" from a dynamically generated string. For instance:

```js
let tag = prompt('ما العلامة التي تريد العثور عليها؟', 'h2');

let regexp = new RegExp(`<${tag}>`); // مثل /<h2>/ أذا أُجيب "h2" في prompt أعلاه
```

## الأعلام

التعبيرات المنتظمة ربما تحتوي علي أعلام تؤثر في البحث.

هناك فقط 6 منهم في ال JavaScript:

`pattern:i`
: عند أستخدام هذا العلم. البحث لا يهتم بحالة الاحرف. فلا أختلاف بين ال `A` وال `a` (أنظر للمثال بالأسفل).

`pattern:g`
: باستخدام هذه العلامة ، يبحث البحث عن جميع التطابقات ، بدونها -- يتم إرجاع المطابقة الأولى فقط

`pattern:m`
: وضع متعدد الاسطر (تم تغطيتها في الفصل <info:regexp-multiline-mode>).

`pattern:s`
: يمكننا وضع ال "dotall", يسمح النقطة `pattern:.` لمطابقة السطر الجديد `\n` (تم تغطيته في الفصل <info:regexp-character-classes>).

`pattern:u`
: Enables full Unicode support. The flag enables correct processing of surrogate pairs. More about that in the chapter <info:regexp-unicode>.

`pattern:y`
: وضع ال "Sticky": يبحث عن الموضع في النص (تم تغطيتها في الفصل <info:regexp-sticky>)

```smart header="الألوان"
من هنا في نظام الألوان:

- التعبير المنتظم -- `pattern:red`
- النص (حيث نبحث) -- `subject:blue`
- النتيجة -- `match:green`
```

## البحث بأستخدام: str.match

كما ذكرنا سابقًا ، يتم دمج التعبيرات المنتظمة مع وظائف النص.

الوظيفة `str.match(regexp)` تجد كل ما يحتوي علي `regexp` في النص `str`.

لديها 3 طرق عمل:

1. أذا كان التعبير المنتظم يحتوي علي العلم `pattern:g`, تقوم بإرجاع مجموعة من كافة التطابقات:

   ```js run
   let str = 'We will, we will rock you';

   alert(str.match(/we/gi)); // We,we (مصفوفة من نصين متطابقين)
   ```

   من فضلك انتبه ان كلا من ال `match:We` وال `match:we` تم أيجادهم, لان العلم `pattern:i` يجعل التعبير المنتظم لا يهتم بحالة الاحرف.

2. إذا لم يكن هناك مثل هذا العلم ، فإنه يُرجع المطابقة الأولى فقط في شكل مصفوفة ، مع المطابقة الكاملة في الفهرس `0` وبعض التفاصيل الإضافية في الخصائص:

   ```js run
   let str = 'We will, we will rock you';

   let result = str.match(/we/i); // بدون العلم g

   alert(result[0]); // We (المتطابقة الاولي)
   alert(result.length); // 1

   // االتفاصيل:
   alert(result.index); // 0 (موضع الجزء الذي تم البحث عنه)
   alert(result.input); // We will, we will rock you (النص)
   ```

   قد تحتوي المصفوفة على فهارس أخرى ، إلى جانب `0` إذا تم تضمين جزء من التعبير المنتظم بين قوسين. سنتناول ذلك في الفصل <info:regexp-groups>.

3. وأخيراُ, أذا كان هناك ولا متطابقة, فأن الناتج يكون `null` (لا يهم أذا كان يوجد العلم `pattern:g` او لا يوجد).

   هذا فارق بسيط مهم للغاية. إذا لم تكن هناك تطابقات ، فلن نتلقى مصفوفة فارغة ، ولكن بدلاً من ذلك نتلقى `null`. قد يؤدي نسيان ذلك إلى حدوث أخطاء ، على سبيل المثال:

   ```js run
   let matches = 'JavaScript'.match(/HTML/); // = null

   if (!matches.length) {
     // خطأ: لا يمكن قراءة الخاصية 'length' لدي null
     alert('خطأ في السطر الأعلي');
   }
   ```

   إذا أردنا أن تكون النتيجة دائمًا مصفوفة ، فيمكننا كتابتها بهذه الطريقة:

   ```js run
   let matches = "JavaScript".match(/HTML/)*!* || []*/!*;

   if (!matches.length) {
     alert("لا متطابقات"); // الأن أنها تعمل
   }
   ```

## الأستبدال: str.replace

الوظيفة `str.replace(regexp, replacement)` تستبدل المتطابقات التي يتم أيجادها بأستخدام ال `regexp` في النص `str` مع كلمة `replacement` (كل المتطابقات يتم أستبدالها أذا كان هنالك العلم `pattern:g`, غير ذلك, يتم أستبدال المتطابقة الاولي فقط).

علي سبيل المثال:

```js run
// بدون العلم g
alert('We will, we will'.replace(/we/i, 'I')); // I will, we will

// مع أستخدام العلم g
alert('We will, we will'.replace(/we/gi, 'I')); // I will, I will
```

العنصر الثاني يكون النص `replacement`. يمكننا استخدام تركيبات أحرف خاصة لإدراج أجزاء من المتطابقة:

| الرموز               | الاجراء في نص الأستبدال                                                                              |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| `$&`                 | إدراج المتطابقة كاملة                                                                                |
| <code>$&#096;</code> | أدراج جزء من النص قبل المتطابقة                                                                      |
| `$'`                 | أدراج جزء من النص بعد المتطابقة                                                                      |
| `$n`                 | أذا `n` تكون رقم او رقمين, ثم يدرج محتويات الأقواس رقم n ، المزيد عنها في الفصل <info:regexp-groups> |
| `$<name>`            | يدرج محتويات الأقواس مع "الاسم" المحدد ، المزيد عنه في الفصل <info:regexp-groups>                    |
| `$$`                 | أدراج حرف `$`                                                                                        |

مثال علي `pattern:$&`:

```js run
alert('أنا أحب HTML'.replace(/HTML/, '$& and JavaScript')); // أنا أحب HTML and JavaScript
```

## الأختبار: regexp.test

الوظيفة `regexp.test(str)` يبحث عن تطابق واحد على الأقل, أذا وُجد, يكون الناتج او العائد `true`, غير ذلك `false`.

```js run
let str = 'أنا أحب JavaScript';
let regexp = /أحب/i;

alert(regexp.test(str)); // true
```

لاحقًا في هذا الفصل ، سندرس تعبيرات أكثر انتظامًا ، وسنتعرف على المزيد من الأمثلة ، ونلتقي أيضًا بطرق أخرى.

يتم توفير معلومات كاملة عن الأساليب في المقالة <info:regexp-methods>.

## الملخص

- يتكون التعبير المنتظم من نمط وأعلام اختيارية: `pattern:g`, `pattern:i`, `pattern:m`, `pattern:u`, `pattern:s`, `pattern:y`.
- بدون أعلام او رموز خاصة (ذلك سوف ندرسه لاحقاً), البحث عن طريق regexp هو نفسه البحث عن سلسلة فرعية.
- الوظيفة `str.match(regexp)` يبحث عن المتطابقات: كلها إن وجدت `pattern:g` علم, غير ذلك, فقط المتطابقة الاولي.
- الوظيفة `str.replace(regexp, replacement)` تستبدل المتطابقات التي توجد بأستخدام `regexp` مع `replacement`: كلهم أن تم أيجادهم `pattern:g` علم, غير ذلك الاولي فقط.
- الوظيفة `regexp.test(str)` تُعيد لنا `true` أذا كان هناك علي الاقل متطابقة واحدة, غير ذلك, تُعيد لنا `false`.

# Assembly Reversing Writeup

## Exercise: Patchpot ->
המשימה לגרום לקובץ ההרצה InjectMe.exe להציג תיבת הודעה (MessageBox) עם השם שלי, לפני שהקוד המקורי ממשיך כרגיל.
הקוד המוזרק לא אמור לשנות את זרימת התוכנית – לאחר סגירת תיבת ההודעה שלי, הביצוע צריך להמשיך כמתוכנן.

#### השלבים:

1. **זיהוי הפונקציה sub_4014D3 כאחראית להדפסת ה־MessageBox המקורי**

זו הפונקציה שמבצעת בפועל את קריאת MessageBoxA ובגללה ה־MessageBox המקורי מוצג.
אחרי אותה קריאה הופיעו שורות ריקות (NOP), שהעידו על מקום פנוי.
   
2. **הוספת קוד חדש (MyInjectedCode) באזור ה־NOPים**

בחלק של האזור הפנוי (בחרתי בכתובת 004010B9) יצרתי את הלייבל MyInjectedCode:

![4](https://github.com/shirelsan/Assembly-Reversing/blob/main/4.png?raw=true)  

3. **הזזת המחרוזות החדשות לתחום פנוי מעבר ל־NOPs**

את המחרוזות שמרתי במקום נפרד, החל מהכתובת 00401100:
![4](https://github.com/shirelsan/Assembly-Reversing/blob/main/5.png?raw=true)  
4. **ריסת הקריאה המקורית בקפיצה לקוד החדש**

בכתובת 004010B0 החלפתי את הקריאה המקורית:
``` call    sub_4014D3 ```
ב- 
``` jmp     MyInjectedCode```

5. **השלמת הקוד עם Continue_as_usual**
   
בסיום MyInjectedCode הוספתי קפיצה חזרה:
```asm
jmp     Continue_as_usual    ; 004010B5
```
הכתובת Continue_as_usual הוגדרה:
```asm
004010B5 Continue_as_usual: ; המשך הקוד המקורי
    nop
    nop
    nop
    nop
```
כך התוכנית חוזרת בדיוק להמשך שהייתה אחרי call sub_4014D3.

6. **הרצת התוכנית:**
   
![4](https://github.com/shirelsan/Assembly-Reversing/blob/main/6.png?raw=true) 

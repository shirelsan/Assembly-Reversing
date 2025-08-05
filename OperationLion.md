### Assembly Reversing WriteUp - 
# CTF Summary Exercise - Operation Lion 

Submitted by: Shirel Sananes 322328814
________________________________________________________________________________

### שלב 1: נטרול מערכת נ"מ

### **מטרה**

לנטרל סוללת נ"מ ולהגן על טייסים, תוך כדי הרצת הקובץ, הודפסו הודעות שמדגישות שישנם "פיתיונות" של האויב, וצריך לאתר את המטרה האמיתית במהירות.
הודעת שגיאה בסיום השלב הייתה: Danger! Anti aircraft system is still operating

### ניתוח הקוד:
#### 1. הבנת מבנה הפונקציה main: 
בתוך main, מתבצעים השלבים הבאים:

1. הדפסות ועיצוב טקסטואלי של הודעת השלב.
2. הפונקציה sub_401290 מדפיסה תווים באפקט איטי - שינוי שם הפונקציה בקוד שלי: Print_char_by_char
3. שליפת שני ערכים מהזיכרון הקבוע:
   
• משתנה ראשון: dword_405474 = 0x61646152 (מחרוזת 'R' 'a' 'd' 'a')
   
• משתנה שני: word_405478 = 0x72 (תו 'r')
   
← שילוב הערכים נותן את המחרוזת "Radar"

שני הערכים מאוחסנים בזיכרון המקומי (stack) ומועברים כמצביע לפונקציה sub_4013B0 (שינוי שם שלי בקוד: Xor2A_Double_NoEffect)

• ניתוח sub_4013B0:

הפונקציה מבצעת עיבוד על המחרוזת שהועברה לה:
1. מחושבת האורך של המחרוזת באמצעות לולאה שמחפשת את תו הסיום (\0)
2. לאחר מכן יש לולאה שמבצעת פעמיים XOR 0x2A על כל תו

המשמעות של הפעולה:

```c
for (i = 0; i < strlen(str); i++) {
    str[i] ^= 0x2A;
    str[i] ^= 0x2A;
}
```
מכיוון ש־XOR עם אותו ערך פעמיים מחזיר את הערך המקורי, הפונקציה לא משנה דבר בפועל. היא ככל הנראה נועדה לבלבל.

• ניתוח sub_401330:
  1. מאתחלת משתנה לערך '{' (0x7B)
  2. מריצה לולאה 5 פעמים:
```c
v = (i * 7) ^ v;
```
בסוף הלולאה, התוצאה היא v = 2
  
  3. קוראת לפונקציות הבאות:

### ניתוח פונקציית sub_4012E0:
הפונקציה מקבלת פרמטר מספרי (`arg_0`) ומחזירה ערך בהתאם לשארית החלוקה ב־3:

- אם שארית החלוקה היא 0 ← מחזירה `arg_0 * 2`
- אם שארית החלוקה היא 1 ← מחזירה `arg_0 + 100`
- אם שארית החלוקה היא 2 ← מחזירה `arg_0 - 50`

כלומר, עבור הקלט 2, הפונקציה מחזירה 48-.

 ### ניתוח הפונקציה sub_3C1430:
 
הפונקציה בודקת האם קיום קובץ בנתיב מסוים ומגיבה בהתאם לתוצאה.

#### מהלך הפעולה:

1. הפונקציה יוצרת את הנתיב: `C:\ReversingCTF\DroneAttack.txt`

2. לאחר מכן, מבוצעת בדיקה האם הקובץ קיים בעזרת הקריאה `FindFirstFileA`.

3. אם הקובץ **נמצא**:
- מודפסת ההודעה: `"Anti aircraft system located Intiating disable sequence..."`
- מתבצע ניסיון לפתוח את הקובץ עם CreateFileA בשביל להמשיך.

4. אם הקובץ **לא נמצא**:
- מודפסת ההודעה: `"Danger! Anti aircraft system is still online!"`

### **פתרון**

לצורך קבלת הודעת ההצלחה בשלב זה, יש לוודא כי הקובץ הבא קיים: `C:\ReversingCTF\DroneAttack.txt`

יצירת הקובץ באופן ידני איפשרה התקדמות לשלב הבא:
```sql
Anti aircraft system located
Initiating disable sequence
Great job. Anti aircraft system is disabled
Stage 2: You are a jet fighter pilot. The sky is clear...
```
בנוסף, נוצר לי קובץ חדש בשם: `AttackIRGC.dll`  וישמש אותי בהמשך – ככל הנראה בשלב 2.

### שלב 2: 
* מסקנת בניים: AttackIRGC.dll לא DLL תקני, הפורמט של PE מחייב להתחיל ב־MZ. (כלומר 0x4D 0x5A) ולי זה מתחיל ב־: 
0F 15 DD 42 41 ... 


#### פענוח קובץ ה-DLL (AttackIRGC.dll):

• פתיחה ב־CFF Explorer ו־Hex Editor הראתה כי הקובץ אינו מתחיל בחתימת MZ, כמצופה מקובץ PE תקין ואין Entry point שמעיד על קובץ DLL.

* חישוב XOR על התווים הראשונים של הקובץ מאפשר גילוי מפתח ההצפנה:
* הבייט הראשון בקובץ: 0x0F -אמור להיות 0x4D ('M')
←  חישוב: 0x0F ^ 0x4D = 0x42 (התו 'B')
* הבייט השני בקובץ: 0x15 - אמור להיות 0x5A ('Z')
←  חישוב: 0x15 ^ 0x5A = 0x4F (התו 'O')

בנוסף, באזור ה־Hex קיימת חזרתיות ברורה של הרצף "BOMB", מה שמרמז על מפתח ההצפנה.

קוד פייתון לפענוח הקובץ ויצירת קובץ מפוענח:

![CODE](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_CTF/code_py.png?raw=true)

**תוצאה:**

מפתיחת הקובץ המפוענח Decoded.dll ניתן לראות כי הקובץ תקין, עם כל החלקים והמבנה הצפוי של קובץ DLL תקין, ולא רק תצוגת Hex-Edior.


![CODE](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_CTF/dll.png?raw=true)

### תהליך ניתוח קובץ Decoded.dll:

1. מבדיקה ב-Export Directory ראיתי שהקובץ מכיל פונקציה בשם hack_security פתחתי את הקובץ DLL בIDA וחיפשתי אחרי הפונקציה על מנת להבין את החתימה שלה והאם היא מקבלת פרמטרים.
2. מבדיקת הפונקציה ראיתי שהיא מקבלת פרמטר 1 (arg_0). בנוסף, היא גם בודקת אם הפרמטר שהיא מקבלת שווה ל-0x2008:
```asm
cmp     [ebp+arg_0], 2008h
jz      short loc_100014A3
```
אם כן, היא קופצת ל־loc_100014A3 ומודפס מסר הצלחה. אם לא מודפס מסר שגיאה. 

3. **כתיבת קוד לייבוא הDLL וקריאה לפונקציה hack_security עם הפרמטר 0x2008:**

![CODE](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_CTF/func.png?raw=true)

4. הרצת התוכנית הובילה להדפסה:
    "Pilot, mission failed. Return to base"
  
לכן נסיתי לחקור את הפונקציה וכשמתי breakpoint על השורה של ההשוואה בשביל לראות את התנאי הbreakpoint לא עבד והתוכנית נסגרה. הבנתי שככה"נ יש פה מנגנון אנטי-דיבג.

5. **ניתוח DllMain:**
* הפונקציה בודקת אם fdwReason == 1 כלומר האם ה־DLL נטען.
*  אם כן, היא קוראת לפונקציה sub_10001200.
*  אם הפונקציה לא מחזירה 1, מודפס "Pilot, mission failed..." ואז ```asm call ExitProcess ```

  כלומר, התהליך נסגר לפני שאני מגיעה ל־hack_security
  
6. **הבנת הפונקציה sub_10001200 ולגרום לה להחזיר תמיד 0:**
  * הפונקציה מחזירה האם התהליך נמצא תחת Debugger. אם כן ← BeingDebugged == 1 אם לא ← BeingDebugged == 0
  * כיוון שמדובר במנגנון אנטי-דיבג לפי הוראות מותר לבצע פיצפוץ' לכן שיניתי את הפונקציה שתיראה כך:
```asm
mov eax, 0
ret
```
![CODE](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_CTF/nop.png?raw=true)
והרצת התוכנית שוב הובילה להדפסה:
```sql
Great job pilot, bombs hit IRGC.

Stage 3: Welcome cyber specialist.
Your mission : Penetrate the security system of the supreme leader.
The location of the enriched Uranium is stored there.
Your country depends on your skills. We COUNT on you. Good luck.
Enter code
```
### שלב 3:



### ניתוח הפונקציה sub_3C1390:

* חתימת הפונקציה: int sub_3C1390(int a, int b)
```asm
mov     eax, [ebp+arg_0]     ; eax = a
xor     eax, [ebp+arg_4]     ; eax = a ^ b

mov     ecx, [ebp+arg_0]     
and     ecx, [ebp+arg_4]     ; ecx = a & b
add     eax, ecx             ; eax = (a ^ b) + (a & b)

mov     edx, [ebp+arg_0]     
or      edx, [ebp+arg_4]     ; edx = a | b
sub     eax, edx             ; eax = ((a ^ b) + (a & b)) - (a | b)
retn
```
בפועל הפונקציה מחשבת: result = (A ^ B) + (A & B) - (A | B) שזה שווה ← return (a + b) - (a | b)

כלומר, הפונקציה מחזירה את ההפרש בין סכום שני המספרים לבין תוצאת פעולת OR עליהם.

המשמעות: כמה ביטים מיותרים נוספו ב־OR לעומת סכום מדויק.

ייתכן שהמטרה היא להסתיר פעולה פשוטה מתוך ניסיון להקשות על רברסינג.









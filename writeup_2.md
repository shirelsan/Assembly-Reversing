# Assembly Reversing Writeup

This document contains explanations and solutions for various Assembly reversing exercises.  
Each section includes the challenge goal, reverse engineering process, and the final solution.

---
## Exercise: Wonderland 2

#### תיאור כללי

הפונקציה מקבלת מחרוזת קלט (`Buffer`), מבצעת עליה עיבוד XOR בבלוקים של 4 בתים עם מפתח קבוע (`0x41524241`), ומשווה את התוצאה למחרוזת מטרה קבועה ("into the rabbit hole").
אם המחרוזת המפוענחת תואמת למחרוזת המטרה, מתבצעת קפיצה לאזור קוד המאשר הצלחה.


#### שלבי פעולת הקוד:

* **מצב התחלתי -בדיקת דגל**
  
בתחילת הקוד, נטען הערך `1` לרגיסטר `eax`, כהגדרת דגל הצלחה כברירת מחדל.
לאחר מכן מתבצעת הפקודה `test eax, eax`, שבודקת אם `eax` שווה ל־0 (באמצעות AND עם עצמו) פעולה זו משפיעה על דגל האפס (ZF-Jump if Zero) מבלי לשנות את תוכן הרגיסטר.
אם הערך היה 0, פקודת הקפיצה `jz` הייתה מעבירה את הביצוע לכתובת `loc_401427`, שם מתבצעת סיום הפונקציה (`mov esp, ebp`, `pop ebp`, `retn`).
מכיוון ש־`eax` שווה ל־1, הקפיצה אינה מתבצעת והקוד ממשיך לרוץ כרגיל.

* **ניקוי הזיכרון (memset)**

הפונקציה memset ממומשת כאן כ־Thunk – כלומר, הפנייה ישירה לפונקציה החיצונית __imp_memset באמצעות פקודת jmp.
הפונקציה עצמה ממלאת אזור בזיכרון בערך קבוע על פי שלושה פרמטרים:

* ערך (Val) , מספר הבתים למילוי (Size),  כתובת ההתחלה (Buffer)

הקריאה לפונקציה מתבצעת לפי מוסכמת הקריאה cdecl, כך שהפרמטרים נדחפים למחסנית בסדר הבא:


  ```asm
  push    400h            ; Size – מספר הבתים למילוי (1024)
  push    0               ; Val – הערך למילוי (0)
  lea     ecx, [ebp+Buffer]
  push    ecx             ; ptr – מצביע לתחילת אזור הזיכרון
  call    memset
  ```

  בקריאה זו, הפונקציה תבצע מילוי של 1024 בתים מהכתובת `Buffer` בערך 0x00, וכך מאפסת את הזיכרון לפני קריאת הקלט.
  
  * **קריאת מחרוזת מהקלט (fgets)**
  
  לאחר מכן, הפונקציה קוראת מחרוזת מהקלט אל המשתנה `Buffer` באמצעות `fgets`, עם אורך מקסימלי של 1023 תווים (0x3FF).
  ומתבצעת קריאה לפונקציה `__acrt_iob_func` לקבלת מצביע לקלט, ואז `fgets` קוראת את הטקסט ל־Buffer.

  * **חישוב אורך המחרוזת**

 האורך של המחרוזת שנקלטה נשמר במשתנה מקומי (`var_C`) באמצעות קריאה ל־`strlen`

  * **לולאת עיבוד XOR** - מבוצעת לולאה החוזרת על מחרוזת הקלט בבלוקים של 4 בתים:
    
  1. בכל איטרציה, ארבעת הבתים ב־Buffer עוברים XOR עם מפתח קבוע 0x41524241.
  2. התוצאה נשמרת בחזרה במיקום המתאים ב־Buffer.
  3. הלולאה ממשיכה כל עוד i + 3 < strlen(Buffer).
  
  * **השוואת מחרוזות**
  
  לאחר סיום הפענוח, מבוצעת השוואת מחרוזת בין התוצאה המפוענחת (Buffer) לבין המחרוזת `"into the rabbit hole"`.
  ההשוואה מתבצעת באמצעות `strncmp`.
  אם אורך המחרוזת שווה (`eax == 0`), מתבצעת קפיצה לחלק שמדפיס שהסיסמה נכונה (`loc_401413`).

  * **פונקצית סיום -**
    בסיום מתבצעת פעולת ניסוי המחסנית הקריאות (`mov esp, ebp` ו־`pop ebp`) ואז `retn`.
    
---
#### **פתרון - פענוח באמצעות סקריפט פייתון**

כדי לפתור את שלב 2 ולמצוא את הסיסמה הנכונה, היה צריך לבצע את הפעולה ההפוכה לזו שבוצעה בתוכנית —

כלומר, **לבצע XOR הפוך** על המחרוזת `"into the rabbit hole"`, בעזרת סקריפט Python -
הפונקציה xor_decrypt מבצעת XOR על המחרוזת עם אותו מפתח, ומחזירה את המחרוזת המפוענחת.

להלן הסקריפט שבוצע:

```python
def xor_decrypt(target: bytes, key: int) -> bytes:
    # Create a mutable byte array to store the decrypted result
    result = bytearray()
    for i in range(0, len(target), 4):          # Process the input data in 4-byte blocks
        block = target[i:i+4]

        while len(block) < 4:                   # Pad the block with zeros if it's less than 4 bytes
            block += b'\x00'
        # Convert the 4-byte block to an integer (little-endian format)
        block_val = int.from_bytes(block, byteorder='little')
        decrypted_val = block_val ^ key          # XOR

        # Convert the decrypted integer back to 4 bytes
        decrypted_block = decrypted_val.to_bytes(4, byteorder='little')
        result += decrypted_block
    return result[:len(target)]

target_string = b"into the rabbit hole"
key = 0x41524241  
decrypted = xor_decrypt(target_string, key)

print("Password:")
print(decrypted.decode('ascii', errors='replace')) 
```
* הסבר על הפונקציה:

הפונקציה xor_decrypt מפענחת מחרוזת בתים (bytes) על ידי שימוש במפתח XOR בן 4 בתים (32 ביט).
הפונקציה מחלקת את המחרוזת לבלוקים של 4 בתים, מבצעת על כל בלוק פעולת XOR עם המפתח, ומחזירה את התוצאה המפוענחת באותו אורך כמו הקלט.
אם הבלוק האחרון קצר מ-4 בתים, מתווספים אפסים כדי להשלים אותו.

הסיסמה שקיבלנו: **(,&.a6:$a03##+&a)->$**



## Exercise: Wonderland 3


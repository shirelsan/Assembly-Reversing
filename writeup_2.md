# Assembly Reversing Writeup

This document contains explanations and solutions for various Assembly reversing exercises.  
Each section includes the challenge goal, reverse engineering process, and the final solution.

---
## Exercise: Wonderland 2

בתחילת השלב יש קטע שבודק את ערכו של רגיסטר eax. בתחילה, מועבר הערך 1 אל eax — כלומר, מוגדר דגל הצלחה כברירת מחדל. 
לאחר מכן מבוצעת פקודת test eax, eax — פקודה שבודקת האם הערך ברגיסטר eax הוא אפס או לא, באמצעות AND ביטויי בין eax לעצמו. פקודה זו משפיעה על דגל האפס (ZF) מבלי לשנות את תוכן eax.

לאחר מכן, פקודת הקפיצה המותנית jz (Jump if Zero) תקפוץ למיקום loc_401427 אם דגל האפס מוגדר, כלומר אם תוצאת ה־test הייתה אפס (כלומר eax שווה ל־0). בכתובת loc_401427 נמצא בלוק הסיום של הפונקציה, שבו מתבצע ניקוי מחסנית הקריאות על ידי שחזור ערכי esp ו־ebp, ולאחר מכן פקודת retn שמחזירה את הבקרה לפונקציה שקראה.

מאחר שבתחילת התוכנית eax שווה ל־1, הקפיצה לא מתבצעת והקוד ממשיך לפעול ברצף לאחר פקודת ה־test.


הפונקציה memset מוגדרת כ־Thunk, כלומר הפניה ישירה (jmp) לפונקציה החיצונית __imp_memset. הפונקציה ממלאת אזור בזיכרון בערך קבוע בהתאם לפרמטרים המועברים לה.
קריאה לפונקציה מתבצעת לפי מוסכמת הקריאה cdecl, באמצעות דחיפת הפרמטרים ל־stack בסדר הבא:
```asm
push    400h            ; Size – number of bytes to fill (1024)
push    0               ; Val – the value to fill with (0)
lea     ecx, [ebp+Buffer]
push    ecx             ; ptr – pointer to the start of the memory area
call    memset
```
בקריאה זו, הפונקציה תבצע מילוי של 1024 בתים מהכתובת Buffer בערך 0x00.





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
print(decrypted.decode('ascii', errors='replace'))  # 'replace' replaces invalid bytes with '?'
```
<u>הסבר על הפונקציה:</u>

הסבר על הפונקציה:
הפונקציה xor_decrypt מבצעת פענוח של מחרוזת בתים (bytes) על ידי שימוש במפתח XOR בן 4 בתים (32 ביט).
הפונקציה מחלקת את המחרוזת לבלוקים של 4 בתים, מבצעת על כל בלוק פעולת XOR עם המפתח, ומחזירה את התוצאה המפוענחת באותו אורך כמו הקלט.
אם הבלוק האחרון קצר מ-4 בתים, הוא מתווסף באפסים כדי להשלים אותו.

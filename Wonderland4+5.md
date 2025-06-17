# Assembly Reversing Writeup - Wonderland Levels 4 & 5

Submitted by: Shirel Sananes 322328814

## Exercise: Wonderland 4

### **מטרה**

להזין קלט שיגרום לתוכנית לעבור את שתי בדיקות:

1. השוואת תוכן - הקלט צריך להתאים למחרוזת "Good luck!!"
2. השוואת כתובת - הכתובת שמכניסים צריכ להתאים לכתובת בזיכרון שבה נמצאת המחרוזת. 

### ניתוח הקוד:
1. **סוג קלט** - זיהוי הקלט המצופה בחלק הזה בקוד:
  ```asm
push    offset aDu      ; " %du"
call    scanf
  ```
 התוכנית מצפה לקלט מסוג unsigned decimal - כלומר, המספר שאני מכניסה הוא עשרוני. 
 
 2. **בדיקת התוכן** -

  ```asm
call    ds:strncmp
  ```

התוכנית משווה בין הקלט שהוזן (Str1) לבין המחרוזת שהורכבה ב־stack היא "Good luck!!".

3. **בדיקת הכתובת** -
```asm
cmp [ebp+Str1], lea [ebp+Str]
```
יש השוואה של הכתובת של הקלט לכתובת המחרוזת בזיכרון. כלומר, התוכנית בודקת אם הקלט (שאמור להיות כתובת) שווה לכתובת שבה מאוחסנת המחרוזת שהורכבה.

### שלבי הפתרון:

1. הרצתי את התוכנית ב־debugger, עם נקודות עצירה, עצרתי לפני הקריאה ל־strncmp.בשביל לבדוק מה הכתובת של המחרוזת שהורכבה ב־stack - של str:

![1](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_wonder4+5/wonder4_1.jpg?raw=true) 

2. הכנסתי את הערך הדיצימלי של הכתובת 0x19fefc -> 1703676 אבל קיבלתי את המסר:

Cheater.. Try again another way.

4. חיפשתי בזיכרון כתובת אחרת שבה מופיעה אותה מחרוזת:

![2](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_wonder4+5/wonder4.jpg?raw=true) 

מצאתי כתובת קבועה בזיכרון שבה המחרוזת קיימת - בתמונה קפיצה לכתובת 0x40473e:

![3](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_wonder4+5/wonder4_2.jpg?raw=true) 

לכן המרת הכתובת 0x40473e למספר עשורני צריך העבור את שתי הבדיקות גם של התוכן וגם של הכתובת. 

### **הפתרון: 0040473e -> 4212542**

![4](https://github.com/shirelsan/Assembly-Reversing/blob/main/image_wonder4+5/sol4.jpg?raw=true) 

## Exercise: Wonderland 5

### **מטרה**

### **ניתוח הקוד**

### **שלבי הפתרון**

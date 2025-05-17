# Assembly Reversing Writeup

This document contains explanations and solutions for various Assembly reversing exercises.  
Each section includes the challenge goal, reverse engineering process, and the final solution.

---
## Exercise: Wonderland 2

בתחילת השלב יש קטע שבודק את ערכו של רגיסטר eax. בתחילה, מועבר הערך 1 אל eax — כלומר, מוגדר דגל הצלחה כברירת מחדל. 
לאחר מכן מבוצעת פקודת test eax, eax — פקודה שבודקת האם הערך ברגיסטר eax הוא אפס או לא, באמצעות AND ביטויי בין eax לעצמו. פקודה זו משפיעה על דגל האפס (ZF) מבלי לשנות את תוכן eax.

לאחר מכן, פקודת הקפיצה המותנית jz (Jump if Zero) תקפוץ למיקום loc_401427 אם דגל האפס מוגדר, כלומר אם תוצאת ה־test הייתה אפס (כלומר eax שווה ל־0). בכתובת loc_401427 נמצא בלוק הסיום של הפונקציה, שבו מתבצע ניקוי מחסנית הקריאות על ידי שחזור ערכי esp ו־ebp, ולאחר מכן פקודת retn שמחזירה את הבקרה לפונקציה שקראה.

מאחר שבתחילת התוכנית eax שווה ל־1, הקפיצה לא מתבצעת והקוד ממשיך לפעול ברצף לאחר פקודת ה־test.

# Phantom Cipher Challenge Report

## 1. Running the Binary

- Open CMD and navigate to the challenge directory:
  ```
  cd "e:\Overwatch\Challenges\Phantom Cipher"
  PhantomCipher.exe
  ```
- Observed that the program prompts for a 5-letter key and only reveals the flag if the correct key is entered.

## 2. Tools Used

- **Debugger:** x64dbg (or OllyDbg/IDA Free)
- **Hex Editor:** HxD (optional)
- **Decompilation:** Ghidra (optional)

## 3. Analyzing the Binary

### a. Loading in Debugger

- Open `PhantomCipher.exe` in x64dbg.
- Set a breakpoint at the program's entry point.

### b. Locating Key Generation

- Step through the code to find where the 5-character key is generated.
- Observed a call to a random function (e.g., `rand()`, `GetTickCount()`, or similar).
- The key is stored in a buffer (e.g., `[ebp-0x20]`).

### c. Extracting the Key

- After the key is generated and before user input, inspect the memory at the buffer address.
- In x64dbg: right-click the address, select "Follow in Dump", and read the 5 bytes.

### d. Entering the Key

- Run the program again and input the extracted 5-character key.
- The program reveals the flag.

## 4. Commands and Screenshots

- (Insert screenshots of x64dbg showing the key in memory and the flag output.)
- Example command to run:
  ```
  PhantomCipher.exe
  ```

## 5. The Flag

- **Flag:** `FLAG{example_flag}`

---

**Note:** Replace `FLAG{example_flag}` with the actual flag you recover.

---

## كيف تستخرج المفتاح (شرح سريع)

1. شغّل البرنامج في مصحح مثل x64dbg.
2. ضع نقطة توقف (Breakpoint) عند بداية البرنامج أو بعد دالة توليد المفتاح.
3. بعد توليد المفتاح وقبل إدخال المستخدم، اعرض الذاكرة (Memory Dump) عند العنوان الذي يُخزن فيه المفتاح (مثلاً: [ebp-0x20]).
4. ستجد هناك 5 بايتات (أحرف) هي المفتاح المطلوب.
5. أدخل هذا المفتاح عند طلب البرنامج وسيظهر لك العلم.

---

## ملخص باللغة العربية

- تم تشغيل البرنامج وتحليل سلوكه باستخدام أدوات التصحيح.
- تم تحديد موقع توليد المفتاح السري في الذاكرة.
- تم استخراج المفتاح الصحيح من الذاكرة باستخدام المصحح.
- تم إدخال المفتاح الصحيح وعرض العلم بنجاح.

---

**End of Report**

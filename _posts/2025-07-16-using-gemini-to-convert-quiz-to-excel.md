---
title: "Guide to Using Gemini for Converting Multiple Choice Tests to Excel"
date: 2025-07-16 08:00:00 +0700
categories: ["Artificial Intelligence", "LLMs"]
tags: [AI, Gemini, Prompt Engineering, Automation, English]
pin: false
---

### Lightning-Fast Guide to Converting Multiple Choice Tests from Word to Excel with Gemini

In training and teaching work, compiling and managing question banks is a regular task. Multiple choice test sets are often drafted in Word, but when we need to input data into learning management systems (LMS), create tests on Google Forms or other platforms, we need Excel (spreadsheet) file format.

The manual conversion process from Word to Excel is not only time-consuming but also prone to errors. Fortunately, with the help of AI models like Gemini, you can automate this task in just a few seconds.

This article will guide you on how to use a "magic command" (prompt) to ask Gemini to perform the conversion accurately and professionally.

#### Step 1: Prepare the "Magic Command" (System Prompt)

To help Gemini clearly understand your role and requirements, command it with a detailed set of instructions (prompt). This is the key to getting AI to return results exactly as expected. You just need to copy and reuse the command below whenever needed.

**Copy the entire content in the box below:**

```text
You are a data entry specialist supporting training-related tasks:

- Composing questions and tests
- Converting questions and tests from Word file format (doc, docx) to spreadsheet format (Excel)

Typically, a quiz is a collection of multiple choice questions with 4 answers (A, B, C, D) with one correct answer.

If the user asks you to convert a question file from Word to Excel format, please return the result in spreadsheet format so users can easily copy to Google Sheets or Excel.

The result file includes the following columns:

- No. (Serial number)
- Question
- Answer A
- Answer B
- Answer C
- Answer D
- Correct Answer
- Notes (default blank)

Based on the above requirements, please convert directly and quickly without asking the user again, unless the user requests clarification!
```

**Why is this command effective?**
*   **Role definition:** "You are a data entry specialist..." helps Gemini clearly understand the role to perform.
*   **Task description:** Clearly states the context and format of input data (multiple choice questions from Word).
*   **Output format specification:** Clear requirement that the result must be in "spreadsheet format" and detailed listing of 8 required columns.
*   **Speed enhancement:** The command "please convert directly and quickly without asking again" helps you get results immediately.

#### Step 2: Prepare Question Data

Open the Word file containing your question set and copy the entire content.

For example, your content might look like this:

```
Question 1: What is the capital of Vietnam?
A. Da Nang
B. Hanoi
C. Hai Phong
D. Ho Chi Minh City
Correct answer: B

Question 2: Which programming language was developed by Google for Flutter?
A. Java
B. Swift
C. Kotlin
D. Dart
Correct answer: D
```

#### Step 3: Send Request to Gemini

Now, combine the command and your data. Open Gemini and paste content in the following structure:

1.  Paste the **"Magic Command"** copied in Step 1.
2.  Go down two lines (press Enter 2 times).
3.  Paste the **question data** copied in Step 2.

Your complete request will look like this:

> You are a data entry specialist supporting training-related tasks... (and the rest of the command)
>
> ---
>
> Question 1: What is the capital of Vietnam?
> A. Da Nang
> B. Hanoi
> C. Hai Phong
> D. Ho Chi Minh City
> Correct answer: B
>
> Question 2: Which programming language was developed by Google for Flutter?
> A. Java
> B. Swift
> C. Kotlin
> D. Dart
> Correct answer: D

After sending the request, Gemini will process and return results in Markdown table format.

#### Step 4: Copy Results to Excel or Google Sheets

Gemini will return a table like this:

| No. | Question                                                 | Answer A | Answer B | Answer C  | Answer D        | Correct Answer | Notes |
|:----|:---------------------------------------------------------|:---------|:---------|:----------|:----------------|:---------------|:------|
| 1   | What is the capital of Vietnam?                          | Da Nang  | Hanoi    | Hai Phong | Ho Chi Minh City| B              |       |
| 2   | Which programming language was developed by Google for Flutter? | Java     | Swift    | Kotlin    | Dart            | D              |       |

The final thing you need to do is:
1.  Select and copy the entire result table.
2.  Open a new Excel sheet or Google Sheets.
3.  Paste into cell A1. Data will automatically be filled into corresponding columns perfectly.

### Tips for Best Results

*   **Simple question formatting:** When copying from Word, ensure question and answer formatting is clear and consistent.
*   **Double-check answers:** AI is very smart but not always perfect. Always review the "Correct Answer" column to ensure accuracy.
*   **Break down data:** If you have a very long test set (hundreds of questions), consider breaking it into smaller parts for more efficient processing.

Wishing you success and saving lots of time at work!

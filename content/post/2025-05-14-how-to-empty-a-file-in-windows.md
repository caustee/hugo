---
date: "2025-05-14T00:00:00Z"
title: "How to empty a file in Microsoft Windows without deleting it"
---
---


So today I had an interesting challange, because I had to fix a space issue on a Windows server. And the issue was caused by a very large text file.

![wow that's a large text file](/images/44gb.jpg)

I know the Linux way of emptying a file which is to redirect STDIN to the file.

But on Windows I had to look and find. Funny thing, I've asked Gemini 2.0 Flash and it was a complete failure. I've tested it locally and it managed to break my code. More about this below.

So, if you're looking for a fix, this is your solution: [Clear-Content module in PowerShell](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/clear-content?view=powershell-7.5). Yet another case when [Stackoverflow](https://superuser.com/a/1017856) was better than current AI models.

In order to test and validate the command before going to Prod server, I had to create a simple app that would write to a file every sec, just to simulate the error file that was constantly written by the app:
```
# Simulate an application writing to a log file
$logFilePath = "TestLog.txt"
# Create the file if it doesn't exist
if (-not (Test-Path $logFilePath)) {
    New-Item -Path $logFilePath -ItemType File | Out-Null
}
$counter = 0
while ($true) {
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logEntry = "[$timestamp] Application is doing some work. Counter: $counter"
    Add-Content -Path $logFilePath -Value $logEntry
    Write-Host "Writing: $logEntry"
    Start-Sleep -Seconds 1
    $counter++
}
```

Run this in Powershell and it will reply with something like this:
```
Writing: [2025-05-14 11:55:37] Application is doing some work. Counter: 1781
Writing: [2025-05-14 11:55:38] Application is doing some work. Counter: 1782
Writing: [2025-05-14 11:55:39] Application is doing some work. Counter: 1783
```

In another powershell window I've used the Clear-Content module to empty the file:
```
PS C:\Users\Administrator> Clear-Content .\TestLog.txt
```

I've issued this command multiple times to make sure it works and I don't catch a corner-case where both apps are using the file.

The command and result was almost instantly, including on the +44GB file from Prod.


Now, if you're curious, here is what Gemini 2.0 Flash LLM proposed:

----

>*I have a running MSSQL on a Windows 10 Server. This SQL server has some issues and is writting a lot of error messages into a SQLDump10000 file. This file reached 44GB in size. How can I clean it quickly without impacting the application?
I know that in Linux this will require a simple redirect using `> SQLDump10000.txt` but I don't know the equivalent on Windows.*

That's a hefty dump file\! You're right, on Linux, redirecting output is a common way to truncate a file. While Windows doesn't have the exact same redirection behavior for immediate truncation of an open file, here are a few approaches you can take to clear the `SQLDump10000` file on your Windows Server without impacting your running SQL Server application:

**Important Considerations Before Proceeding:**

  * **Ensure sufficient disk space:** Even a temporary copy or move operation will require free space equal to the file size.
  * **Understand the root cause:** While clearing the file will free up disk space, it's crucial to investigate *why* these dump files are being created in the first place. Addressing the underlying SQL Server issues will prevent this from happening again. Check your SQL Server error logs for more details.
  * **Consider archiving:** Before deleting, you might want to archive the dump file to a different location for later analysis if needed.

**Methods to Clear the SQLDump File:**

Here are a few methods you can use, ordered roughly by their potential impact and complexity:

**1. Using `fsutil file setzerolength` (Recommended - Minimal Impact):**

This command is designed to set the length of a file to zero, effectively emptying it without deleting and recreating it. This is generally the safest approach as it doesn't involve deleting or moving the file while SQL Server might still have it open.

  * Open **Command Prompt as Administrator**.

  * Navigate to the directory containing the `SQLDump10000` file.

  * Execute the following command, replacing the filename if yours is different:

    ```
    fsutil file setzerolength SQLDump10000
    ```

  * Verify that the file size has been reduced to 0 bytes.

**2. Using PowerShell (Alternative - Minimal Impact):**

PowerShell offers another way to achieve a similar result.

  * Open **PowerShell as Administrator**.

  * Navigate to the directory containing the `SQLDump10000` file using `cd`.

  * Execute the following command:

    ```powershell
    (Get-Item SQLDump10000).truncate(0)
    ```

  * Verify that the file size has been reduced to 0 bytes.

**3. Moving the File (Slightly Higher Risk):**

You can move the large file to another location. This is generally safe as long as the SQL Server process doesn't have an exclusive lock on the file handle for writing.

  * Open **File Explorer**.
  * Navigate to the directory containing the `SQLDump10000` file.
  * Right-click on the file and choose **Cut**.
  * Navigate to a different drive or folder with sufficient space and choose **Paste**.
  * Once the file is moved, a new, empty `SQLDump10000` file might be created by SQL Server if the error condition persists.

**4. Deleting the File (Potentially Disruptive - Use with Caution):**

Deleting the file is the most straightforward approach but carries a slightly higher risk if SQL Server is actively trying to write to it at the exact moment of deletion.

  * Open **File Explorer**.
  * Navigate to the directory containing the `SQLDump10000` file.
  * Right-click on the file and choose **Delete**.
  * Confirm the deletion.
  * SQL Server will likely create a new `SQLDump10000` file if it continues to encounter errors.

**Why the Linux `>` redirection doesn't directly translate to Windows for open files:**

In Linux, the `>` redirection typically opens the file in write-only mode and truncates it to zero length *before* the command on the left side of the redirection is executed. If a process has the file open for writing, this truncation can still often succeed.

Windows handles file locking and sharing differently. If SQL Server has an exclusive or shared write lock on the `SQLDump10000` file, a simple redirection attempt using something like `echo. > SQLDump10000` from the command prompt might fail or not immediately truncate the file in the way you expect, especially if the application is actively writing to it.

**Recommendation:**

I highly recommend using the **`fsutil file setzerolength`** command as it's designed for this purpose and has the least potential to interfere with the running SQL Server process. PowerShell's `truncate()` method is also a good alternative.

After clearing the file, remember to investigate the root cause of the SQL Server errors to prevent future large dump files. Check your SQL Server error logs and consider reviewing recent changes or configurations.

----
>*The powershell option sounds good. I want to test this locally before doing it on Prod env.
Give me a small program that will run on windows (could be a ps script that just reads/writes this file every 1s). I just want to make sure that my powershell truncate command will not break the actual app.*

That's a smart and cautious approach\! Testing locally is definitely the right way to go. Here's a small PowerShell script that simulates an application continuously writing to a file named `TestLog.txt` every second. You can run this script and then, in another PowerShell window, try the `truncate()` command.

```powershell
# Simulate an application writing to a log file

$logFilePath = "TestLog.txt"

# Create the file if it doesn't exist
if (-not (Test-Path $logFilePath)) {
    New-Item -Path $logFilePath -ItemType File | Out-Null
}

$counter = 0
while ($true) {
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logEntry = "[$timestamp] Application is doing some work. Counter: $counter"
    Add-Content -Path $logFilePath -Value $logEntry
    Write-Host "Writing: $logEntry"
    Start-Sleep -Seconds 1
    $counter++
}
```

**How to use this script:**

1.  **Save the code:** Open a text editor (like Notepad) and paste the code into it. Save the file as `WriteLog.ps1` (or any other `.ps1` extension).
2.  **Run the script:**
      * Open **PowerShell as Administrator**.
      * Navigate to the directory where you saved `WriteLog.ps1` using the `cd` command.
      * Execute the script by typing `.\WriteLog.ps1` and pressing Enter.
      * You will see messages being written to the console every second, and a file named `TestLog.txt` will be created (or updated) in the same directory.

**Now, in a separate PowerShell window (also run as Administrator):**

1.  Navigate to the same directory where `TestLog.txt` is being written.

2.  Execute the truncate command:

    ```powershell
    (Get-Item TestLog.txt).truncate(0)
    ```

3.  Observe what happens in both PowerShell windows and check the `TestLog.txt` file.

**Expected Behavior:**

  * The `truncate()` command in the second PowerShell window should successfully set the size of `TestLog.txt` to 0 bytes almost instantly.
  * The first PowerShell script should continue to run without throwing any immediate errors. It will likely continue writing new log entries to the now empty `TestLog.txt` file.

**What this test demonstrates:**

This test simulates a scenario where another process (your SQL Server) has the file open and is actively writing to it. By successfully truncating the file using PowerShell's `truncate()` method while the writing process is ongoing, you can gain confidence that this approach is likely safe for your SQL Server dump file.

**To stop the `WriteLog.ps1` script:**

In the PowerShell window where the script is running, press `Ctrl + C`.

Let me know how your local test goes\! If you encounter any issues or have further questions, feel free to ask.

----
>ok the method is great. the script writes something every second. But the clear script doesn't work as it should:
>``` (Get-Item ../TestLog.txt).truncate(0)```
>and this is the output:
```
PS C:\Users\Administrator\Desktop> .\empty_log.ps1
Method invocation failed because [System.IO.FileInfo] does not contain a method named 'truncate'.
At C:\Users\Administrator\Desktop\empty_log.ps1:1 char:1
+ (Get-Item ../TestLog.txt).truncate(0)
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
    + FullyQualifiedErrorId : MethodNotFound
```
>the TestLog.txt file is in the homedir of Administrator user.

Ah, you are absolutely right\! My apologies for the oversight in the previous script. The `System.IO.FileInfo` object in PowerShell doesn't directly have a `truncate()` method like some other programming languages.

Thank you for catching that and providing the error output. It's crucial to be precise, especially when dealing with production systems.

Here's the corrected PowerShell command to truncate the file:

```powershell
$file = Get-Item ../TestLog.txt
$file.OpenWrite().SetLength(0)
$file.Close()
```

**Explanation of the corrected command:**

1.  **`$file = Get-Item ../TestLog.txt`**: This part correctly retrieves the `FileInfo` object for your `TestLog.txt` file located one directory level up from where you are likely running the `empty_log.ps1` script (assuming `empty_log.ps1` is in your Desktop folder).
2.  **`$file.OpenWrite()`**: This opens the file for writing. This method returns a `System.IO.FileStream` object.
3.  **`.SetLength(0)`**: This method of the `FileStream` object sets the length of the file to 0 bytes, effectively truncating it.
4.  **`$file.Close()`**: This closes the file stream, releasing the lock on the file. While it might work without explicitly closing it in this simple scenario, it's good practice to close file streams when you're done with them.

**How to use the corrected command:**

1.  Make sure your `WriteLog.ps1` script is still running and writing to `TestLog.txt`.

2.  Open a **new PowerShell window as Administrator**.

3.  Navigate to the directory where you are executing the truncate command (likely your Desktop if `empty_log.ps1` is there, or one level up if you want to target `../TestLog.txt`).

4.  Execute the corrected commands one by one or together on a single line separated by semicolons:

    ```powershell
    $file = Get-Item ../TestLog.txt; $file.OpenWrite().SetLength(0); $file.Close()
    ```

Now, when you run this corrected command while the `WriteLog.ps1` script is running, you should see the `TestLog.txt` file being emptied without interrupting the writing process of the first script.

Again, I apologize for the error in the previous response and appreciate you pointing it out. Let me know if this corrected version works as expected for you\!

----
>* still didn't work:
```
PS C:\Users\Administrator\Desktop> .\empty_log.ps1
Method invocation failed because [System.IO.FileInfo] does not contain a method named 'Close'.
At C:\Users\Administrator\Desktop\empty_log.ps1:3 char:1
+ $file.Close()
+ ~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
    + FullyQualifiedErrorId : MethodNotFound
```
>And this broke the app:
```
Writing: [2025-05-14 11:21:15] Application is doing some work. Counter: 207
Add-Content : The process cannot access the file 'C:\Users\Administrator\TestLog.txt' because it is being used by another process.
At C:\Users\Administrator\Desktop\sim_app_write_log.ps1:14 char:5
+     Add-Content -Path $logFilePath -Value $logEntry
+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : WriteError: (C:\Users\Administrator\TestLog.txt:String) [Add-Content], IOException
    + FullyQualifiedErrorId : GetContentWriterIOError,Microsoft.PowerShell.Commands.AddContentCommand
``

You are absolutely correct again\! It seems I'm having a bit of trouble with the `FileInfo` object methods today. My apologies for the continued errors.

The issue is that `$file.OpenWrite()` returns a `FileStream` object, which has the `SetLength()` and `Close()` methods, not the original `FileInfo` object.

Here's the corrected script to empty the log file:

```powershell
$logFilePath = "../TestLog.txt"
$stream = [System.IO.FileStream]::new($logFilePath, [System.IO.FileMode]::OpenOrCreate, [System.IO.FileAccess]::Write, [System.IO.FileShare]::ReadWrite)
$stream.SetLength(0)
$stream.Close()
```

**Explanation of this corrected command:**

1.  **`$logFilePath = "../TestLog.txt"`**: Defines the path to your log file.
2.  **`$stream = [System.IO.FileStream]::new(...)`**: This explicitly creates a new `FileStream` object.
      * **`$logFilePath`**: Specifies the path to the file.
      * **`[System.IO.FileMode]::OpenOrCreate`**: Specifies that the file should be opened if it exists, or created if it doesn't.
      * **`[System.IO.FileAccess]::Write`**: Specifies that we intend to write to the file.
      * **`[System.IO.FileShare]::ReadWrite`**: This is crucial\! It allows other processes (like your writing script) to continue reading from or writing to the file while we have it open for truncation.
3.  **`$stream.SetLength(0)`**: This correctly uses the `SetLength()` method of the `FileStream` object to set the file size to 0.
4.  **`$stream.Close()`**: This closes the `FileStream`, releasing our hold on the file.

**Regarding the "app breaking":**

The error you received:

```
Add-Content : The process cannot access the file 'C:\Users\Administrator\TestLog.txt' because it is being used by another process.
```

This indicates that at the exact moment the `Add-Content` command in your writing script was trying to write, the process trying to truncate the file might have had an exclusive lock, even with the previous (incorrect) `OpenWrite()` attempt.

**How to test with the corrected `empty_log.ps1`:**

1.  **Save the following code as `empty_log.ps1` in your Desktop folder:**

    ```powershell
    $logFilePath = "../TestLog.txt"
    $stream = [System.IO.FileStream]::new($logFilePath, [System.IO.FileMode]::OpenOrCreate, [System.IO.FileAccess]::Write, [System.IO.FileShare]::ReadWrite)
    $stream.SetLength(0)
    $stream.Close()
    ```

2.  **Ensure your `sim_app_write_log.ps1` (the writing script) is running.**

3.  **Open a new PowerShell window as Administrator.**

4.  **Navigate to your Desktop folder.**

5.  **Execute the `empty_log.ps1` script:**

    ```powershell
    .\empty_log.ps1
    ```

Now, observe if the `TestLog.txt` file is emptied, and if the `sim_app_write_log.ps1` script continues to run without the "access denied" error. The inclusion of `[System.IO.FileShare]::ReadWrite` in the `FileStream` creation is the key to allowing concurrent access.

I sincerely apologize for the previous incorrect suggestions. Let's hope this corrected version works as expected for your testing\! Please let me know the outcome.

----

> ok, I see this works just as I wanted:
``` Clear-Content .\TestLog.txt ```

Excellent\! I'm glad to hear that the `Clear-Content` cmdlet worked perfectly for your needs in the local test.

You are absolutely correct. `Clear-Content` is a straightforward and effective PowerShell cmdlet specifically designed to remove the content of a file without deleting the file itself. It's often the most direct and PowerShell-idiomatic way to achieve this.

**Key advantages of using `Clear-Content`:**

  * **Simplicity:** It's a single, easy-to-understand command.
  * **PowerShell Native:** It's a built-in cmdlet, so you don't need to work directly with .NET classes unless you have more complex requirements.
  * **Generally Safe:** It's designed to handle file access gracefully.

So, for your production environment, you could certainly use:

```powershell
Clear-Content "Path\To\Your\SQLDump10000"
```

**Important considerations for your production environment:**

  * **Permissions:** Ensure the account running this command has the necessary permissions to modify the `SQLDump10000` file.
  * **Path:** Double-check the exact path to your `SQLDump10000` file.
  * **Scheduling:** If you need to do this regularly, you can schedule this command to run as a scheduled task.

Thank you for exploring the different options and for pointing out the effectiveness of `Clear-Content`. It's a good reminder that PowerShell often has built-in solutions for common tasks.

Do you have any other questions or need further assistance with this or anything else?

---


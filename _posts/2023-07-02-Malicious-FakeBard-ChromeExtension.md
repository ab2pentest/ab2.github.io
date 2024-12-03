---
title:  Malware Analysis - FakeBard - Malicious ChromeExtension
tags: malware malware-analysis cybersecurity phishing malicious chrome extension bard fake 
---

# Description

As it's the weekend and I had some free time, I was scrolling through Facebook when an ad suggestion caught my eye. The ad promoted a new Google Bard version which, in my opinion seems like a potential malware.

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/be073e8d-0765-41f7-8e65-f84e872f1ce1)

Despite my curiosity, I decided to investigate for fun, so I grabbed my laptop and copied the link into my browser.

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/46688f27-26d7-4d3a-9a22-03760695f729)

After downloading the RAR file, I extracted the files inside. The required password, which I obtained from the ad itself, is `888`.

A single file was extracted called `Bard_Setup.msi`

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/158c2480-b52e-4b35-a1fa-4d0f4f3d4551)

# Analyzing process

The internet is full of tools that can extract MSI files without the need to double-click them.

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/240aa5b2-d81e-4e2a-a3a6-47076ed90072)

We can use VirusTotal to search for the SHA256 hash of these files and check if they are detected by any antivirus engines.
This can help us determine if the files are potentially malicious or safe to use.

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/910127b3-2abc-4111-9568-8072bac361cd)

I will only scan the file named `chromedriver.exe`, and it appears to be safe.

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/f88b42ca-0004-4a77-a960-4581ba4f732f)

Now, I can examine the contents of the remaining files by opening them using a text editor or any other preferred method:

1- `setup.bat`:

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/3c0ec32f-24da-4522-949c-f7be6bdbe1c1)

```
    taskkill /F /IM chrome.exe: It forcefully terminates any running instance of the "chrome.exe" process. This effectively closes Google Chrome.

    timeout /t 1 >nul: It introduces a one-second delay in the script execution. The >nul part is used to discard the timeout message displayed on the console.

    start chrome.exe --restore-last-session --load-extension="%~dp0/nmmhkkegccagdldgiimedpiccmgmiedagg4": It starts Google Chrome with two additional command-line arguments. The --restore-last-session argument tells Chrome to restore the last browsing session, while --load-extension="%~dp0/nmmhkkegccagdldgiimedpiccmgmiedagg4" loads a specific Chrome extension located in the same directory as the script (indicated by %~dp0).
```

2- `manifest.json`:

> Every extension requires a JSON-formatted file, named manifest.json, that provides important information. This file must be located in the extension's root directory.

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/280d12e2-7545-419b-b83b-9f8032e7ef3a)

3- `content.js`:

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/c7b4f604-3c2f-4cfe-9c30-d3aa32505dc0)

This code checks if the variable `isContentScriptExecuted` is set in the local storage of the browser.
If the variable is not set, it sends a message to the Chrome runtime using `chrome.runtime.sendMessage()`.
The message contains an object with the property action set to `executeFunction`.
After sending the message, it sets the `isContentScriptExecuted` variable in the local storage to true using `localStorage.setItem()`.
This ensures that the code block inside the if statement will only execute once, as the variable will be set for future checks.

4- `favicon.png`:

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/64b594ea-3491-4840-b5b8-40b5d75c9a7f)

The fake extension utilizes the Google Translate icon.

5- `background.js`:

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/f5d9f4ee-c54d-4d3a-8b0e-916793a6f42c)

This file is obfuscated and packed. I utilized some online tools to deobfuscate the JavaScript code, and here is the result I obtained:

![image](https://github.com/ab2pentest/ab2pentest.github.io/assets/84577967/721c12ff-b351-443c-b742-7abb967fdc95)

I will leverage AI assistance to generate a brief explanation of the file's exact functionality. This approach will save us a considerable amount of time. 😄

```
The code you provided appears to be a JavaScript code snippet. Here's an explanation of what the code is doing:

1-   The code initializes several variables (cook, myList, newLine, dem, vip, demtkqc, dem25, dem50, demtktt, deminr, Liveadstkdie, demnolimit, dem250, nave, se, si, te, exchangeRates, o) and assigns them initial values.

2-    The code defines a function getsss() that returns a promise. Inside the function, it uses the chrome.cookies.getAll() method to retrieve cookies from a specific URL (https://facebook.com). The cookies are then transformed into a string format and resolved as the result of the promise.

3-    The code defines a function GetToken(cook) that returns a promise. Inside the function, it makes a fetch request to retrieve the HTML content of a specific URL (https://www.facebook.com/ads/manager/account_settings/information/). The response HTML is then parsed to extract the token_bm and fb_dtsg values. The extracted values are resolved as the result of the promise.

4-    The code defines a function convertToUSD(amount, currency) that converts an amount value from a given currency to USD. It uses an object called exchangeRates that stores exchange rates for different currencies. If the currency is USD, the function returns the amount unchanged. If the currency is not found in the exchangeRates object, a default value of 1e5 is returned. Otherwise, the function calculates and returns the USD equivalent of the amount based on the exchange rate.

5-    The code calls the getsss() function to retrieve the cookies. If the cookies include the "c_user" property, the code proceeds to retrieve the token and perform further actions.

6-    Inside the callback of the GetToken(cook) promise, the code makes a fetch request to retrieve data about the user's Facebook pages. It uses the token_bm value in the request URL. The response is then parsed, and information about each page (such as ID, verification status, fan count, admin status, and business ID) is extracted and stored in the myList array.

7-    Next, the code makes a fetch request to retrieve data about the user's Facebook businesses. It uses the token_bm value in the request URL. The response is parsed, and information about each business (such as ID, verification status, creation year, and sharing eligibility status) is extracted and stored in the myList array.

8-    Finally, the code makes a fetch request to retrieve data about the user's Facebook ad accounts. It uses the token_bm value in the request URL. The response is parsed, and information about each ad account (such as account ID, account status, currency, payment cycle, amount spent, balance, and business ID) is extracted and stored in the myList array.

9-    The code includes additional logic and calculations related to the extracted data, but it is incomplete in the provided code snippet.

10-   It's worth noting that the code seems to be using some browser-specific functionalities (chrome.cookies.getAll()) and assumes the presence of a navigator object and a fetch() method. It may not work as expected outside of a browser environment.
```

# Conclusion

I believe everything has been thoroughly explained in this quick analysis of the malicious Chrome extension. With the assistance of AI in providing explanations, we have successfully reached the conclusion of this blog post.

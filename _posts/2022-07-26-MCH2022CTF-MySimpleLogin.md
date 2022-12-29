---
title: MCH2022CTF - MySimpleLogin - Writeup
tags: mch2022ctf android apk reverse
---

# Description

![image](https://user-images.githubusercontent.com/84577967/181047281-3ca8b815-7642-4497-8f9c-353f16a00dae.png)

# Solution

We were given an APK file

[MySimpleLogin.zip](https://github.com/ab2pentest/ab2pentest.github.io/files/9191147/MySimpleLogin.zip)

![image](https://user-images.githubusercontent.com/84577967/181047436-7a829280-379b-4711-bc60-42742f8e25fe.png)

After downloading the APK file, we can use any Android DEX decompiler to open it.

To find the entry point or main activity, we can check the `AndroidManifest.xml` file.

![image](https://user-images.githubusercontent.com/84577967/181079155-a0d568ea-ddb7-455e-9972-a4a892de12a3.png)

The main activity for this APK is `ctf.challenges.mysimplelogin.MainActivity`, as shown in the screenshot. We can examine this activity to see what it does.

![image](https://user-images.githubusercontent.com/84577967/181080067-90b3be68-23e9-4421-8cad-a172836803dd.png)

Based on the decompiled code, there is a password checker present in the APK. This function appears to describe how it works.

![image](https://user-images.githubusercontent.com/84577967/181080733-72c304ee-1557-4dc8-ac21-c31003617f00.png)

The function takes an input value `i`, adds it to the string value of `s`, and passes the result to a function called `l`. It then compares the output of `l` to the value of `h`. If they are equal, it calls `showError(w);` if they are not equal, it calls `showFlag(f)`. This seems somewhat illogical, so we should continue reading the code to see if we can find more context or clarification.

[+] The `l` function:

The `l` function appears to be calculating the MD5 hash of the concatenation of `i` and `s`.

![image](https://user-images.githubusercontent.com/84577967/181081555-50cb2c2a-7a4d-4650-a921-02cb378078a3.png)

It's good to know how the `l` function works. To find the values of `s`, `h`, and other string variables, we can check the resources section of the decompiled code.

```
        String s = getResources().getString(R.string.OO0O00OOO00O0O);
        String h = getResources().getString(R.string.OO0O00OOO00OOO);
        String f = getResources().getString(R.string.OO0O0O0OO00OOO);
        String w = getResources().getString(R.string.OO0O0OOOO00OOO);
```

It looks like the values of `s`, `h`, and other string variables are stored in the `strings.xml` file located in the `res/values` directory.

![image](https://user-images.githubusercontent.com/84577967/181082923-6e104e26-22a1-4d41-aa3c-e95ba1692f7b.png)

[+] The `s` value:

![image](https://user-images.githubusercontent.com/84577967/181081326-15c62b1b-50a5-4cfe-b583-4fe216f755d8.png)

[+] The `h` value:

![image](https://user-images.githubusercontent.com/84577967/181083212-07d3231f-180c-44d8-b8cc-70443e77adfe.png)

[+] The `f` value:

Was called in `showFlag(f);`

![image](https://user-images.githubusercontent.com/84577967/181083990-b3bbd6f5-9d22-4de6-a56e-4ea16e7540a2.png)

[+] The `w` value:

was called in `showError(w);`

![image](https://user-images.githubusercontent.com/84577967/181084106-d3129dc2-5996-4883-9a8d-fc33c8a8f3e4.png)

Based on the information we have gathered, it appears that the flag is stored in the `showError(w)` function, not in the `showFlag(f)` function.

![image](https://user-images.githubusercontent.com/84577967/181085016-83d5e086-9a31-42e8-aeb2-0b3238073dba.png)

If we provide the correct input to the app, it looks like it will pass it to the `x` function once and then to the `r` function 7 times.To save time, I immedeatly copied the 3 functions in a new .java file.

**Note: the original value of `w` contains some escaped characeters, so I paste it to this [HTML Entities decoder](https://www.online-toolz.com/tools/text-html-entities-convertor.php) for decoding.**

```java
public class MainActivity{
	
    public static void showError(String e) {
        System.out.println(x(r(r(r(r(r(r(r(e, "r"), "s"), "t"), "u"), "v"), "w"), "x"), "X"));
    }

	public static String r(String s, String c) {
		return s.replace(c, "");
	}

	public static String x(String s, String k) {
		StringBuilder sb = new StringBuilder();
		for (int i = 0; i < s.length(); i++) {
			sb.append((char) (s.charAt(i) ^ k.charAt(i % k.length())));
		}
		return sb.toString();
	}
	
	public static void main(String[] args) {
		showError(">49s?#kjllw>ijvnra;;i>=kuki`ta;`iirj9::xtm;<rij%");
    }
}
```

# Flag

By running the java code above, you should be able to see the flag printed out.

![image](https://user-images.githubusercontent.com/84577967/181085672-5b57ce5a-065f-4ae8-b9d9-679d1bd86521.png)

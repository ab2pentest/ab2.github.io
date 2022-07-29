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

After we download it let's open it in any `Android dex decompiler` ...

First thing is looking for the entrypoint or the main activitiy, this can be found on `AndroidManifest.xml` file

![image](https://user-images.githubusercontent.com/84577967/181079155-a0d568ea-ddb7-455e-9972-a4a892de12a3.png)

Our MainActivity is `ctf.challenges.mysimplelogin.MainActivity` as shown in the screenshot above, so let's go there and check what we have

![image](https://user-images.githubusercontent.com/84577967/181080067-90b3be68-23e9-4421-8cad-a172836803dd.png)

So from the decompiled code it's obvious that we have a password checker and this function explains it

![image](https://user-images.githubusercontent.com/84577967/181080733-72c304ee-1557-4dc8-ac21-c31003617f00.png)

So the function get's our input value defined as `i` and add to it the string value of `s` and pass it to a function called `l` and checks the output of it if equals the value of `h`, in case it's true is going to call `showError(w);` and in case it's false is going to call `showFlag(f);` this looks a bit illogic, let's ignore this now and keep reading the code ...

[+] The `l` function:

This function is just calculating the md5sum of `i+s`

![image](https://user-images.githubusercontent.com/84577967/181081555-50cb2c2a-7a4d-4650-a921-02cb378078a3.png)

Great ! But where is `s` and `h` and all the defined string values ..., well in the decompiled code we can see that they are coming from resources

```
        String s = getResources().getString(R.string.OO0O00OOO00O0O);
        String h = getResources().getString(R.string.OO0O00OOO00OOO);
        String f = getResources().getString(R.string.OO0O0O0OO00OOO);
        String w = getResources().getString(R.string.OO0O0OOOO00OOO);
```

So in the path `res/values/strings.xml` these values were saved

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

Now I confirm that the `showError(w);` is the where our flag is saved and not in `showFlag(f);`, so let's check it

![image](https://user-images.githubusercontent.com/84577967/181085016-83d5e086-9a31-42e8-aeb2-0b3238073dba.png)

So in case we give it the right input it's goint to pass it to the `x` function once and 7 times to `r`, so I didn't waste lot of time and I immedeatly copied the 3 functions in a new .java file

**Note: the original value of `w` has some escaped characeters, so I paste it to this [HTML Entities decoder](https://www.online-toolz.com/tools/text-html-entities-convertor.php)**

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

Just run the script above and it will print out the flag for you

![image](https://user-images.githubusercontent.com/84577967/181085672-5b57ce5a-065f-4ae8-b9d9-679d1bd86521.png)
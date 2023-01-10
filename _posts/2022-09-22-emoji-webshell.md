---
layout: post
title: "PHP Emoji Webshell $🐚"
---

Last time I tried to push myself on learning how to construct a webshell without using any alphabet in PHP. There are a lot of techniques that can be used such as base operation, auto increment, XOR and etc. This [blog](https://websec-readthedocs-io.translate.goog/zh/latest/language/php/webshell.html?_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=zh-CN) have decent techniques that can be used for obfuscation.

![img1][img1]

### Understanding the webshell

The objective of the obfuscation webshell is simple, execute os command without detection. Im assuming the server's configuration is not disabling php functions like `system()`, `passthru()`, `shell_exec()` and other functions that can execute OS command. If the server disabling those function, there will be another topic to bypass the protection. Anyway, our webshell maybe will looks like this one:

- line 3: try to check if `$_POST[4]` is empty or not, if not empty, decode it using base64 and store it in `$a` variable. Otherwise, it'll store `whoami` instead.
- line 4: Execute command from variable `$a`.

Now the crucial part is how to generate alphabet without using alphabet at all. One of my idea is by using XOR and auto increment operation to generate alphabet and numbers, and set it to emoji🥸 as variable. Below are how I use auto increment to generate alphabets.

![img3][img3]

- line 1: Set `$_` variable to an array.
- line 2: Convert array to "Array" in string datatype.
- line 3: `("_" == "_")` is comparing two string, if true it will return 1. `("_" == "_") + ("_" == "_")`, then `(1) + (1)` and becomes 2.
- line 4: `@$_[++$__]` means `@$_[3]`, so it will takes 4th argument from "Array" which is "a".
- line 6-31: set emoji to 'a' and so on. When we increment the variable that contains "a", it will be "b" and so on.

After constructing alphabet, then we can use the certain function like `base64_decode()`, `$_POST[]` and `system()`. Noted that we cannot use [`eval()`](https://www.php.net/manual/en/function.eval.php) as variable since it is a constructor, not a [variable function](https://www.php.net/manual/en/functions.variable-functions.php).

![img4][img4]

- line 1-3: increment the number.
- line 4: set variable `$👿` to "base6".
- line 5-6: decrement the number.
- line 7: construct the word "base64_decode". If you notice, we are using XOR operation to this line to contruct an ascii underscore "_" by xoring hastag with pipe(I would say) `"#" ^ "|"`
- line 9: set variable `$💀` to "system".
- line 10: set variable `$🥳` as "_POST" by using XOR operation to create underscore and capital letter since we do not create capital letter at all.
- line 11: check if `$_POST[4]` is set, if set, it will decode the parameter, otherwise set it to "whoami" to variable `$🤯`.
- line 12: execute `system()` with specified command.

The full source code will be like figure below or this [link](https://github.com/nightfury99/php-emoji-webshell/blob/main/webshell.php).

![img5][img5]

Example for the request for `id` in base64 encoding.

![img6][img6]

So there you go. Just like I said before, there is nothing new with this webshell, but it is good to train your skills. I dont know how relevant this webshell in real life because I just test it at my own lab. Stay curious guys🍻.

[img1]:{{site.baseurl}}/images/2022-09-22/1.png
[img3]:{{site.baseurl}}/images/2022-09-22/3.png
[img4]:{{site.baseurl}}/images/2022-09-22/4.png
[img5]:{{site.baseurl}}/images/2022-09-22/5.png
[img6]:{{site.baseurl}}/images/2022-09-22/6.png
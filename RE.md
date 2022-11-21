sections: 

- [cryptic](#cryptic)
- [key](#key)
- [Reverser](#forwback)

# Cryptic

## The program

when we execute the file, the program asks us to translate some alien text.

![image](https://user-images.githubusercontent.com/89386156/201503837-315eb66e-d0bc-4c2a-8b72-ae2fac85a21b.png)

let's try to translate the alien text through an online translator.

![image](https://user-images.githubusercontent.com/89386156/201503897-af47b5d2-e22d-4284-ba97-9ebba1fe3d6d.png)

hmmm, looks like this isnt the intended way of solving the challenge, lets try analysing the binary file by opening it in radare2

## Initial analysis

![image](https://user-images.githubusercontent.com/89386156/201503923-4ab60fde-0174-42f4-977d-09891282d26e.png)

there are quite a few things to take note of in this assembler dump.

1. this is the part of the program that is used to print out the context and the alien string as seen in the `puts` instruction call.

![image](https://user-images.githubusercontent.com/89386156/201503960-815163cc-8bf0-4fa8-9c38-812dba5b7099.png)

2. a function `sym.str2md5` is called (we will analyse the function later)

![image](https://user-images.githubusercontent.com/89386156/201504004-45b67fb0-2ad1-4ea4-90d9-1cf0d10cfd59.png)

3. `strcmp` is called and a control flow is made here

![image](https://user-images.githubusercontent.com/89386156/201504035-0fb4d341-2937-4e86-a697-f946c8e9055b.png)

## Analysing `sym.str2md5`

this function seems to turn the user's input into md5, and this is confirmed by the imports of modules in `openssl/md5.h` 

![image](https://user-images.githubusercontent.com/89386156/201505266-c81c54eb-9183-44a3-85be-1937bf7c6b08.png)

![image](https://user-images.githubusercontent.com/89386156/201505342-d9ee2858-6d20-4d1b-866a-0bedd2312ef6.png)

md5 is a hash, hence it is irreversable.

with that in mind, the only way to solve this challenge is to bypass the control flow check

## Bypassing the control flow

for the sake of visualisation, I will open the program in graph view and focus on the control flow

![image](https://user-images.githubusercontent.com/89386156/201505431-9eabc08a-de52-4cd5-bd06-44fac49f6906.png)

1. strcmp (string compare) is called
2. the register is checked for a `zero flag`
3. a jump is made if the register does not display a zero flag
4. the jump made if the condition in `3.` holds points towards printing 'wrong', and the jump made if the condition in `3.` fails points towards printing 'correct'

##### *p.s, if you are using MacOs, the condition control flow will look like this: `cbnz {register}, {location of jump}` where `cbnz` refers to `Compare and Branch on Non-Zero`*

### Debugging 

1. open the binary file in radare2's debug mode by suffixing it with a `-d` 

`r2 -d cryptic_{os}` 

2. set a breakpoint at `main` and a breakpoint at the compare statement (the cbnz instruction call on mac).

`db main`

`db {location of the call}` e.g `db 0xaaab69640d84`, where `0xaaab69640d84` points to the location of the compare/ cbnz statement

![image](https://user-images.githubusercontent.com/89386156/201505687-66f63f5d-15f3-465e-9fa4-49fd34946464.png)

3. run the program with `dc`

- radare will tell you that you have hit the breakpoint at main. your `rip` (instruction pointer register) will be at the start of main
- continue with `dc` again

![image](https://user-images.githubusercontent.com/89386156/201505788-3fa1b716-fa44-4e6d-bc58-41e81be02d75.png)

- you have now hit the point where an input is necessary, enter anything and continue with `dc`

4. hitting the control flow branch breakpoint

- we now need to modify the register in use for the comparison to display a zero flag
- do this with the `dr` command

for this example, the `w0` register is being used for the compare statement, we will modify it as so:

![image](https://user-images.githubusercontent.com/89386156/201505877-596e3d03-4dac-43d8-8c62-6121a432275f.png)

`dr w0 = 0`

5. continue and profit

- entering `dc` one more time should print the flag

![image](https://user-images.githubusercontent.com/89386156/201505892-96e62798-5e58-414a-9a75-273163baa779.png)

nailed it. we now have the flag


# key

## The program

when we first execute the file, it prompts us to enter a key

![image](https://user-images.githubusercontent.com/89386156/201512640-53f515c5-cfd8-4b08-b10a-a694c5d6e8c9.png)

and if we enter the key wrongly, it lets us know by printing "nah, try again"

![image](https://user-images.githubusercontent.com/89386156/201512673-f8fdb388-2b5b-484b-b133-4f29797bb958.png)

## Analysis of the program

let's open the file in radare2

I will be using the linux x86_64 version for the sake of analysis. (intel assembly is nicer imo :] )

entering the `main` function, we can see that:

- a function, `arr_ob` is called
- another function, `keeeeey` is called further on

## arr_ob()

lets take a look at `arr_ob`

we can see that at the side, there are letters and symbols which resemble the format of a flag (YBN{...})

![image](https://user-images.githubusercontent.com/89386156/201512863-8ff5f7d1-ed40-4610-98c5-102e58bb784b.png)

we can deduce that this function was used to obfuscate the flag.

## keeeeey()

this function is very complicated, but we'll summarise up the important parts

1. function `strtohex` is called
2. the following part compares a 64 bit register with a hexidecimal value, and returns true/false

![image](https://user-images.githubusercontent.com/89386156/201513217-adc2c2b9-e26d-4958-b03d-a01a4e6a62c4.png)

3. the register (in this case `rax`) is derived from:

![image](https://user-images.githubusercontent.com/89386156/201513365-875a3e47-e3c8-4c79-b950-2fb77a2e2519.png)

the value in `var_40h` now stores the resultant value of `rax*rax + rax`, which is later compared to in the control flow at `2.` 

simply put, if `rax` stores your input, `(rax^2)*2` == `0x731a933c6ca45c82`

### finding the correct input

with this in mind, let's reverse the math of the resultant hex string.

`√(0x731a933c6ca45c82/2)` would result in  `0x79617921`

converting `0x79617921` to text would give us the correct input 

![image](https://user-images.githubusercontent.com/89386156/201513546-31553d52-551c-4afe-8495-7b40dcc81944.png)
(https://www.rapidtables.com/convert/number/hex-to-ascii.html)

indeed, we have got the string `'yay!'`

## Getting the flag

let's run the program again, but this time, we will use the string that we obtained as the key

![image](https://user-images.githubusercontent.com/89386156/201513586-c0b352c6-c507-410c-8f71-3325eb58f89f.png)

challenge solved.

# Forwback

### Writeup credit: [duckupus](https://github.com/Duckupus)

The binary that we get is not stripped of its symbols, allowing us to see the names of the functions.
<br>
The main function looks fairly standard, getting 100 characters of input using `scanf`.

#### NOTE: depending on the linux/mac binary you are reversing, the logic is slightly different. However, it does the same thing.
The `input` first gets modified by some code, before comparision, which looks something like this

```c
for (int i = 0; i < 0x18; i = i + 1) {
	int tmp = 0;
	for (int j = 0; j < 8; j = j + 1) {
			tmp = tmp | (((int)input[i] >> (j & 0x1f) & 1U) != 0) << (7 - j & 0x1f);
	}
	input[i] = tmp;
}
```
This looks complicated, lets break it down a bit!
<br>
First, lets figure out a few things
`i < 0x18` -> This is the length of the entire password it checks. It's the iterator
`j < 8` -> This loop has a oneliner, but loops 8 times, I wonder what happens there?

### Reversing the forback check function, pt. 1
As there's a lot of things happening in that single line, lets break it down into a few portions.
Without worrying too much about the magic that happens inside, we first look at it's output
#### NOTE: `magic` means there is extra code, but it's not the focus right now.
```
tmp = tmp | (magic);
```
we have a `tmp` variable that is ORed each loop(for 8 times), which is then stores the value back into the input, `input[i] = tmp;`.
This input is then checked with a global variable later in the code.
<br>
Now, we have this monster line. Lets work from outside in.

```c
(((int)input[i] >> (j & 0x1f) & 1U) != 0) << (7 - j & 0x1f)
```

### Reversing the forback check function, pt. 2
first,
```c
((magic) != 0) << (7 - j & 0x1f)
```
Notice how whatever is in `magic`, we check if it is `0` or not. Meaning, no matter what `magic` does, we will either get a `1` or `0` output, which is left shifted by
```c
7 - j & 0x1f
```

### Reversing the forback check function, pt. 3
#### NOTE: 0x1f -> 31
now, as `j`, nor `7` will never be bigger then `0x1f`, we can ignore it.
```c
7 - j
```
this means at each loop for `j`, we are comparing `(magic) != 0`, and storing the result in `tmp`, with each iteration storing the result in a lower bit then the previous one.
<br>
Now, lets look at how the final part of the monster line works

### Reversing the forback check function, pt. 4
```c
input[i] >> (j & 0x1f) & 1U
```
This shifts the character bit by `j & 0x1f`, which doesn't do anything. Hence rewritten
```c
(input[i] >> j) & 1
```
this shifts the character to the right by `j` bits, and gets the value at the LSB.

The entire process looks something like this
```
assume input value of
0000 0001 (base2)
output:
1000 0000 (base 2)

input
0100 0000
output
0000 0010
```
Basically, the code flips the bits of the input.

### solving the challenge (forback)
phew! We've came a long way, and now, all that is left is to make the solve script, and hopefully all goes well.

```c
#include <stdio.h>

char input[] = { 0x9a, 0x42, 0x72, 0xde, 0x4e, 0xcc, 0x6e, 0xa6, 0x4e, 0xce, 0x8c, 0x72, 0xe6, 0xb4, 0x2e, 0x16, 0xcc, 0xb4, 0x46, 0x96, 0x2a, 0xce, 0xbe, 0x00 };

int main() {
	for (int i = 0; i < 0x18; i = i + 1) {
		int tmp = 0;
		for (int j = 0; j < 8; j = j + 1) {
			tmp = tmp |
				(((int)input[i] >> (j & 0x1f) & 1U) != 0) << (7 - j & 0x1f);
		}
		input[i] = tmp;
	}
	puts(input);
	return 0;
}
```
Result: `YBN{r3vers1Ng-th3-biTs}`
😉

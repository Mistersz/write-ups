# Write-Up betterthanu

### The following code is given:

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <stdint.h>

FILE *flag_file;
char flag[100];

int main(void) {
    unsigned int pp;
    unsigned long my_pp;
    char buf[16];

    setbuf(stdin, NULL);
    setbuf(stdout, NULL);

    printf("How much pp did you get? ");
    fgets(buf, 100, stdin);
    pp = atoi(buf);

    my_pp = pp + 1;

    printf("Any last words?\n");
    fgets(buf, 100, stdin);
    if (pp <= my_pp) {
        printf("Ha! I got %d\n", my_pp);
        printf("Maybe you'll beat me next time\n");
    } else {
        printf("What??? how did you beat me??\n");
        printf("Hmm... I'll consider giving you the flag\n");
      if (pp == 727) {
            printf("Wait, you got %d pp?\n", pp);
            printf("You can't possibly be an NPC! Here, have the flag: ");

            flag_file = fopen("flag.txt", "r");
            fgets(flag, sizeof(flag), flag_file);
            printf("%s\n", flag);
        } else {
            printf("Just kidding!\n");
        }
    }

    return 0;
}
```


Let´s try to see this program **backwards**:


- We can notice this if statement has a fgets function that will lead us to the flag file.

```C
  if (pp == 727) {
            printf("Wait, you got %d pp?\n", pp);
            printf("You can't possibly be an NPC! Here, have the flag: ");

            flag_file = fopen("flag.txt", "r");
            fgets(flag, sizeof(flag), flag_file);
            printf("%s\n", flag);
        }
```
So we definally need the variable **pp** to be 727.

Scrolling up the code we see:

```C
printf("How much pp did you get? ");
    fgets(buf, 100, stdin);
    pp = atoi(buf);

    my_pp = pp + 1;

    printf("Any last words?\n");
    fgets(buf, 100, stdin);
    if (pp <= my_pp) {
        printf("Ha! I got %d\n", my_pp);
        printf("Maybe you'll beat me next time\n");
    } else {
        printf("What??? how did you beat me??\n");
        printf("Hmm... I'll consider giving you the flag\n");
      if (pp == 727) {
```
<br>
So now we know that in order to enter that if statement and get the flag we need first
guarantee that pp > my_pp (since in the case of pp <= my_pp), we would enter another if statement.


So far so good, so we basically need:
<br>


- Guarantee that pp > my_pp
- pp = 727

The problem is:
We have two fgets calls and both are storing their input in the 16 bytes array called "buf", so
**we can´t directly change the value of my_pp**.

That being the case we will have to OverFlow that buffer, this is possible because the array has the size
of 16 bytes, but we can write 100 bytes to the buffer so it may be possible to corrupt the value of
my_pp and set it to a value lower than 727 to get the flag.

<br>

# Ghidra

Taking a look at the code with Ghidra, we can see where are the variables on the stack:
<br>
<p align="center">
  <img src="https://github.com/Mistersz/Write-Ups/assets/82767252/acfcced0-ce59-44f1-ba12-7b348b15b68c"/>
</p>

We can conclude a few things:

- local_28 is our buffer
- local_c is our pp variable (because it is the variable that recieves the return of atoi function)
- local_18 is our my_pp variable (since it takes local_c and increments by 1)
- local_18 is 16 bytes away from local_28 (and since the buffer grows from the lower address to the highest it´s possible to change local_18!)

With that in mind we can:

1 - Use the first input to set pp = 727.

2 - Use the second input to overflow the buffer and then change my_pp to a value lower than 727.

<br>

# Exploit

For that we can craft a little bufferoveflow exploit with pwntools:

<p align="center">
  <img src="https://github.com/Mistersz/Write-Ups/assets/82767252/d3e06a97-35bc-4ba7-9456-273e40b3ed55"/>
</p>

With this code we are basically:

- Sending a first input of 727
- Sending a payload of 16 0´s and a 0

Since we are writting 16 0´s and a 0 we will overflow the buffer and change the variable my_pp
That way, we will have p = 727, and my_pp = 0

And we get this output:
<p align="center">
  <img src="https://github.com/Mistersz/Write-Ups/assets/82767252/e26e4a6b-ef27-4938-8d73-dcaa682bc299"/>
</p>





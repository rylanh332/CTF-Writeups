# Compiled

This challenge comes with a **Compiled-1688545393558.Compiled** file, which after using the **file** command on it, we can determine that it's an ELF executable file.

Use chmod +x on it in order to make it executable, and we can see that it looks for a password. If we get it wrong it will say "Try again", otherwise, it should give us the flag.

If we use strings on it, we can find some interesting data:

<img width="120" height="66" alt="image" src="https://github.com/user-attachments/assets/aa39e3c0-b93e-420e-b8f1-6e3d6909a574" />

We can see that it has some strings like "DoYouEven%sCTF" and StringsisForNoobs

I also tried using using `ltrace ./Compiled-1688545393558.Compiled` and `strace ./Compiled-1688545393558.Compiled` to get more information. 

If we run `ltrace ./Compiled-1688545393558.Compiled` then we can see that the string is comparing a value

<img width="833" height="133" alt="image" src="https://github.com/user-attachments/assets/eff4921a-9449-4647-bfcd-ffaba42f13fe" />

However, one key detail is the string we found when running `strings` against the executable, specifically "DoYouEven%sCTF." If we rerun the elf and input DoYouEven%sCTF, we will be able to see the variable fill:

<img width="818" height="128" alt="image" src="https://github.com/user-attachments/assets/d8619de6-a55c-4ff5-9bd2-77c33e91a564" />

So this is clearly a portion of the password, and we just need to find what that variable should equal. 

I tried some gdb and got nowhere since the variable we need is not loaded before we view it in the breakpoint.

So instead I used ghidra to decompile the executable and get some code to look at:

<img width="445" height="413" alt="image" src="https://github.com/user-attachments/assets/69c0f6e4-1d96-4662-91c9-5540ecc8a2e9" />

Our two checks are outlined in red, and we can see that our scanf password is set to local_28. local_28 then gets compared to __dso_handle. If our output equals that, then we get hit with Try again! And then if our stripped variable does not equal _init then we fail again.

In one of the earlier ltrace tests, you can see that I input `DoYouEvenTestCTF`, and our output is set to **TestCTF**. This means that all we have to do is input `DoYouEven_init` and we should get "Correct."

And that will solve the challenge.

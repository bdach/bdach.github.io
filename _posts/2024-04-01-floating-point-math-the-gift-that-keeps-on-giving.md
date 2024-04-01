---
layout: post
title: "Floating point math: the gift that keeps on giving"
sub_title: "Or, a treatise on cows not being perfectly spherical."
image:
    path: "/assets/images/lazer/fp2/hero.png"
    thumbnail: "/assets/images/lazer/fp2/thumb.png"
categories: debugging
---

Most of the time, floating point numbers are great. They're fast, they work like what you generally expect math to work, the sun is shining, the birds are singing, everything is right with the world.
And yet, do not be misled: if you're thinking this, you're in a precarious position.
It is a bit like looking at a cow and saying "look at this [perfectly spherical object](https://en.wikipedia.org/wiki/Spherical_cow)!".
At _some_ high enough level you may be close enough with that presumption.
But every now and then you work on something and _irrational_ things start to happen and then you remember just _oh how wrong_ you were before.

Historically, in my debugging war stories, floating point is a common and recurrent perpetrator.
If you've happened to stumble upon this poor excuse of a blog in the past, you may recall [that time I had to reach for the big guns to find out that floating point behaviour is actually configurable via a processor register](/debugging/2020/09/18/when-double-epsilon-can-equal-zero.html) and in some instances may begin to do things that you'd normally call completely unreasonable (and which yet are reasonable - in specific circumstances).
This is a similar story.

# .NET is .NET, Right? Right?

To provide the appropriate context, I write code for the [osu!(lazer)](https://github.com/ppy/osu) project, which is a from-the-ground-up rewrite of the original [osu!](https://osu.ppy.sh/). The original was written in .NET Framework, and the rewrite is in .NET.

Anyone outside of the .NET ecosystem is probably completely baffled right now, so a quick primer: [.NET Framework](https://en.wikipedia.org/wiki/.NET_Framework) was first, and was Windows-only.
After a while, Microsoft decided that it was time to diversify, and made [.NET Core](https://en.wikipedia.org/wiki/.NET), which was multiplatform, and Microsoft saw it was good, so it decided to streamline and renamed .NET Core to .NET and retroactively gave it all the APIs .NET Framework had.
(Except for the fact that not all of them work. Some just throw `NotSupportedException`s. Which is not necessarily bad, mind; let's all keep quiet over the remoting and Code Access Security coffins.)

Now behind the scenes, .NET Framework and .NET, which are supposed to be sorta-kinda compatible and interchangeable, do differ when one digs into the ugly details.
As you can probably guess by now, I was about to find out about one of those.

My goal was to port a specific calculation from the .NET-Framework implementation of osu! to the .NET osu!(lazer) version.
The relevant method looked something like so:

```csharp
internal static (float raw, int rounded) DifficultyPeppyStars(float difficultyHpDrainRate, float difficultyOverall,
    float difficultyCircleSize, int objectCount, int drainLength)
{
    float raw = (difficultyHpDrainRate + difficultyOverall + difficultyCircleSize +
                 Clamp((float)objectCount / drainLength * 8, 0, 16)) / 38 * 5;
    int rounded = (int)Math.Round(raw);
    return (raw, rounded);
}

internal static float Clamp(float value, float min, float max)
{
    if (Single.IsNaN(min) || Single.IsNaN(max))
    {
        return Single.NaN;
    }

    Debug.Assert(max >= min);

    if (value > max)
    {
        return max;
    }

    if (value < min)
    {
        return min;
    }

    return value;
}
```

so I mindlessly wrote the following in .NET instead:

```csharp
public static (float raw, int rounded) DifficultyPeppyStars(float drainRate, float overallDifficulty, float circleSize, int objectCount, int drainLength)
{
    var objectToDrainRatio = Math.Clamp((float)objectCount / drainLength * 8, 0, 16);
    
    float raw = (drainRate
                + overallDifficulty
                + circleSize
                + objectToDrainRatio) / 38 * 5;

    int rounded = (int)Math.Round(raw);
    return (raw, rounded);
}
```

the only notable difference being the usage of `Math.Clamp()` which made its way into the standard library.

I don't particularly want to do a deep cut as to what that's calculating, because it's *mostly* irrelevant, so I'll attempt to keep to the relevant bits: I needed this bit of code to produce the *exact same* result on osu!(lazer) as it did on osu!.
If it didn't, it would break an algorithm that was supposed to estimate maximum achievable scores on osu! beatmaps (for the uninitiated: that's a loose equivalent of a level in the game), and thus return complete nonsense.

Now the issue is, that both methods above - given the exact same parameters - would return a *different* result on .NET Framework than it did on .NET.
Which was a telltale sign that whatever caused this was going to be fun to look into.

# I Know I'm Not Crazy, I Know It Rounds These Numbers Differently

In a way, .NET is a "two-tiered" environment, if you will.
The programmers write C# (or F#, if you're one of the cool kids, or Visual Basic.NET, if you are a poor soul suffering through eternal damnation).
The compiler spits out a bytecode called the Common Intermediate Language, which is then JITted to actual native code at execution time.

Thus the reasoning goes, that if the C# code is the same, but its result is not, then either the emitted CIL changed, or the JITted code did.
I checked the CIL first - and wouldn't you know it, it was _mostly_ the same.
.NET Framework did this:

```
  .method assembly hidebysig static valuetype [mscorlib]System.ValueTuple`2<float32, int32>
    DifficultyPeppyStars(
      float32 difficultyHpDrainRate,
      float32 difficultyOverall,
      float32 difficultyCircleSize,
      int32 objectCount,
      int32 drainLength
    ) cil managed
  {
    .param [0]
      .custom instance void [mscorlib]System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[])
        = (
          01 00 02 00 00 00 03 72 61 77 07 72 6f 75 6e 64 // .......raw.round
          65 64 00 00                                     // ed..
        )
        // string[2]
          /*( string('raw') string('rounded') )*/
    .maxstack 4
    .locals init (
      [0] int32 rounded
    )

    // [50 13 - 51 87]
    IL_0000: ldarg.0      // stack: [difficultyHpDrainRate]
    IL_0001: ldarg.1      // stack: [difficultyOverall, difficultyHpDrainRate]
    IL_0002: add          // stack: [difficultyOverall + difficultyHpDrainRate]
    IL_0003: ldarg.2      // stack: [difficultyCircleSize, difficultyOverall + difficultyHpDrainRate]
    IL_0004: add          // stack: [difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_0005: ldarg.3      // stack: [objectCount, difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_0006: conv.r4      // stack: [(float)objectCount, difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_0007: ldarg.s      // stack: [drainLength, (float)objectCount, difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_0009: conv.r4      // stack: [(float)drainLength, (float)objectCount, difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_000a: div          // stack: [(float)objectCount / drainLength, difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_000b: ldc.r4       8
    IL_0010: mul          // stack: [(float)objectCount / drainLength * 8, difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_0011: ldc.r4       0.0
    IL_0016: ldc.r4       16
    IL_001b: call         float32 Clamp(float32, float32, float32)
    IL_0020: add          // stack: [Clamp((float)objectCount / drainLength * 8, 0, 16) + difficultyCircleSize + difficultyOverall + difficultyHpDrainRate]
    IL_0021: ldc.r4       38
    IL_0026: div
    IL_0027: ldc.r4       5
    IL_002c: mul

    // [52 13 - 52 48]
    IL_002d: dup
    IL_002e: conv.r8
    IL_002f: call         float64 [mscorlib]System.Math::Round(float64)
    IL_0034: conv.i4
    IL_0035: stloc.0      // rounded

    // [53 13 - 53 35]
    IL_0036: ldloc.0      // rounded
    IL_0037: newobj       instance void valuetype [mscorlib]System.ValueTuple`2<float32, int32>::.ctor(!0/*float32*/, !1/*int32*/)
    IL_003c: ret


  } // end of method DifficultyPeppyStars
```

and .NET did this:

```
  .method public hidebysig static valuetype [System.Runtime]System.ValueTuple`2<float32, int32>
    DifficultyPeppyStars(
      float32 drainRate,
      float32 overallDifficulty,
      float32 circleSize,
      int32 objectCount,
      int32 drainLength
    ) cil managed
  {
    .param [0]
      .custom instance void [System.Runtime]System.Runtime.CompilerServices.TupleElementNamesAttribute::.ctor(string[])
        = (
          01 00 02 00 00 00 03 72 61 77 07 72 6f 75 6e 64 // .......raw.round
          65 64 00 00                                     // ed..
        )
        // string[2]
          /*( string('raw') string('rounded') )*/
    .maxstack 3
    .locals init (
      [0] float32 objectToDrainRatio,
      [1] int32 rounded
    )

    // [43 9 - 43 90]
    IL_0000: ldarg.3      // objectCount
    IL_0001: conv.r4
    IL_0002: ldarg.s      drainLength
    IL_0004: conv.r4
    IL_0005: div
    IL_0006: ldc.r4       8
    IL_000b: mul
    IL_000c: ldc.r4       0.0
    IL_0011: ldc.r4       16
    IL_0016: call         float32 [System.Runtime]System.Math::Clamp(float32, float32, float32)
    IL_001b: stloc.0      // objectToDrainRatio

    // [45 9 - 48 52]
    IL_001c: ldarg.0      // drainRate
    IL_001d: ldarg.1      // overallDifficulty
    IL_001e: add
    IL_001f: ldarg.2      // circleSize
    IL_0020: add
    IL_0021: ldloc.0      // objectToDrainRatio
    IL_0022: add
    IL_0023: ldc.r4       38
    IL_0028: div
    IL_0029: ldc.r4       5
    IL_002e: mul

    // [50 9 - 50 44]
    IL_002f: dup
    IL_0030: conv.r8
    IL_0031: call         float64 [System.Runtime]System.Math::Round(float64)
    IL_0036: conv.i4
    IL_0037: stloc.1      // rounded

    // [51 9 - 51 31]
    IL_0038: ldloc.1      // rounded
    IL_0039: newobj       instance void valuetype [System.Runtime]System.ValueTuple`2<float32, int32>::.ctor(!0/*float32*/, !1/*int32*/)
    IL_003e: ret

  } // end of method
```

Well it looks completely fine. The only difference is that .NET computes the `objectToDrainRatio` first, but otherwise I didn't see any real room for error there, so it was time to examine the native code.
For a test call of `DifficultyPeppyStars(5f, 9.5f, 3.7f, 1820, 250)`, .NET Framework executed this:

```
d94634         fld     dword ptr [esi+34h]         % @esi+34h = 00 00 A0 40 = 5f
                                                   % st0= 5.00000000000000000000e+0000 (0:4001:a000000000000000)
d84638         fadd    dword ptr [esi+38h]         % @esi+38h = 00 00 18 41 = 9.5f
                                                   % st0= 1.45000000000000000000e+0001 (0:4002:e800000000000000)
d84630         fadd    dword ptr [esi+30h]         % @esi+30h = CD CC 6C 40 = 3.7f
                                                   % st0= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
db86f8000000   fild    dword ptr [esi+0F8h]        % @esi+F8h = 1C 07 00 00 = 1820i
                                                   % st0= 1.82000000000000000000e+0003 (0:4009:e380000000000000)
                                                   % st1= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
d95df8         fstp    dword ptr [ebp-8]           % @ebp-8 = 00 00 00 00
                                                   % @ebp-8 = 00 80 E3 44 = 1820f
                                                   % st0= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
d945f8         fld     dword ptr [ebp-8]           % just loads back what was just spilled from stack...
db86f0000000   fild    dword ptr [esi+0F0h]        % @esi+0F0h = FA 00 00 00 = 250i
                                                   % st0= 2.50000000000000000000e+0002 (0:4006:fa00000000000000)
                                                   % st1= 1.82000000000000000000e+0003 (0:4009:e380000000000000)
                                                   % st2= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
d95df8         fstp    dword ptr [ebp-8]           % see above
d945f8         fld     dword ptr [ebp-8]           % see above
def9           fdivp   st(1), st
                                                   % st0= 7.28000000000000024869e+0000 (0:4001:e8f5c28f5c28f800)
                                                   % st1= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
d80dd81d050b   fmul    dword ptr ds:[0B051DD8h]
                                                   % st0= 5.82400000000000019895e+0001 (0:4004:e8f5c28f5c28f800)
                                                   % st1= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
83ec04         sub     esp, 4
d91c24         fstp    dword ptr [esp]
6a00           push    0
6800008041     push    41800000h
dd5df0         fstp    qword ptr [ebp-10h]
e814e49515     call    209B01C8                    % this is the clamp call, everything before is just setup
dd45f0         fld     qword ptr [ebp-10h]
                                                   % st0= 1.82000000476837158203e+0001 (0:4003:919999a000000000)
                                                   % st1= 1.60000000000000000000e+0001 (0:4003:8000000000000000)
dec1           faddp   st(1), st
                                                   % st0= 3.42000000476837158203e+0001 (0:4004:88ccccd000000000)
d835e01d050b   fdiv    dword ptr ds:[0B051DE0h]
                                                   % st0= 9.00000001254834591791e-0001 (0:3ffe:e666666bca1af000)
d80de81d050b   fmul    dword ptr ds:[0B051DE8h]
                                                   % st0= 4.50000000627417318100e+0000 (0:4001:900000035e50d800)
                                                   % result ever so slightly above 4.5!
dd5df0         fstp    qword ptr [ebp-10h]
dd45f0         fld     qword ptr [ebp-10h]
db5df8         fistp   dword ptr [ebp-8]
                                                   % @ebp-8 = 05 00 00 00
```

and .NET executed this:

```
c5fa118524010000     vmovss  dword ptr [rbp+124h], xmm0
c5fa108534010000     vmovss  xmm0, dword ptr [rbp+134h]           % xmm0=           0            0            0            5
c5fa588524010000     vaddss  xmm0, xmm0, dword ptr [rbp+124h]     % xmm0=           0            0            0         14.5
c5fa118520010000     vmovss  dword ptr [rbp+120h], xmm0
488b8d08020000       mov     rcx, qword ptr [rbp+208h]
49bbf075de07fd7f0000 mov     r11, 7FFD07DE75F0h
ff15604e6dff         call    qword ptr [7FFD07DE75F0h]
48898518010000       mov     qword ptr [rbp+118h], rax
488b8d18010000       mov     rcx, qword ptr [rbp+118h]
3909                 cmp     dword ptr [rcx], ecx
e8533e78fd           call    00007FFD05E965F8
c5fa118514010000     vmovss  dword ptr [rbp+114h], xmm0           % xmm0=           0            0            0          3.7
c5fa108520010000     vmovss  xmm0, dword ptr [rbp+120h]
c5fa588514010000     vaddss  xmm0, xmm0, dword ptr [rbp+114h]     % xmm0=           0            0            0         18.2
c5fa118510010000     vmovss  dword ptr [rbp+110h], xmm0
c5f857c0             vxorps  xmm0, xmm0, xmm0
c5fa2a85f8010000     vcvtsi2ss xmm0, xmm0, dword ptr [rbp+1F8h]   % xmm0=           0            0            0         1820
c5e857d2             vxorps  xmm2, xmm2, xmm2
c5ea2a95f4010000     vcvtsi2ss xmm2, xmm2, dword ptr [rbp+1F4h]   % xmm2=           0            0            0          250
c5fa5ec2             vdivss  xmm0, xmm0, xmm2                     % xmm0=           0            0            0         7.28
c5fa59050f030000     vmulss  xmm0, xmm0, dword ptr [7FFD08712AF8h]% xmm0=           0            0            0        58.24
c5fa10150b030000     vmovss  xmm2, dword ptr [7FFD08712AFCh]
c5f057c9             vxorps  xmm1, xmm1, xmm1
e8ee2978fd           call    00007FFD05E951E8
c5fa11850c010000     vmovss  dword ptr [rbp+10Ch], xmm0           % xmm0=           0            0            0           16
c5fa108510010000     vmovss  xmm0, dword ptr [rbp+110h]
c5fa58850c010000     vaddss  xmm0, xmm0, dword ptr [rbp+10Ch]     % xmm0=           0            0            0         34.2
c5fa5e05e6020000     vdivss  xmm0, xmm0, dword ptr [7FFD08712B00h]% xmm0=           0            0            0          0.9
c5fa5905e2020000     vmulss  xmm0, xmm0, dword ptr [7FFD08712B04h]% xmm0=           0            0            0          4.5
                                                                  % xmm0=00000000 00000000 00000000 40900000
                                                                  % this is a perfect 4.5!

c5fa5ac0             vcvtss2sd xmm0, xmm0, xmm0                   % promotion to double
e84d2678fd           call    00007FFD05E94E78                     % this is the round call
c5fb118500010000     vmovsd  qword ptr [rbp+100h], xmm0
c5fb2c8d00010000     vcvttsd2si ecx, mmword ptr [rbp+100h]        % @rbp+100h = 00 00 00 00 00 00 10 40 = 4d
                                                                  % ecx=4
```

What you'll notice here is that while the intermediate results of the operations can be sort of matched to each other, the two runtimes use different instruction families for the floating point computations.
.NET Framework uses x87, while .NET uses SSE instructions.

x87 is quite old now. It started its life as an extension of x86, and it entered the scene with Intel's 8087 floating-point coprocessor in 1980.
Comparatively, SSE is the "new" kid on the block, introduced almost 2 decades later, boasting performance gains due to enabling SIMD.
However, the other thing that separates those two, and the kicker here, is that x87 registers are _higher-precision_.
They're _80 bits_ wide, with 64 bits for the mantissa.
Meanwhile, while SSE/AVX registers can be technically wider (now up to 512 bits), the way they work is that each register is split into multiple words, and every instruction is performed on each word _independently_.
Each of those words can be at most _64 bits_, period.
And in this case it's even worse, `xmm0` is a 128-bit register, split into four _32-bit_ words.

This explains the discrepancy; .NET's results were technically _wronger_ than .NET Framework's, because of precision loss.
And even if they weren't, .NET Framework was where my ground truth was, so that's what I had to match anyway.

Well, okay, so that's that conundrum solved, but the thing is, how do you fix it? osu!(lazer) is cross-platform.
Newer architectures won't even _have_ an x87-compatible processor.
How does one fix this there?
Am I gonna have to ship an x87 software emulator for this?

# And Then I Got A Wonderful, Awful Idea

Confession time: I am maybe not the laziest person around, but there is a considerable amount of laziness in me.
And implementing a x87 software emulator was considerably above my personal line of "can't be bothered".
(That, and I also anticipated the bewildered reaction of reviewers when they were gonna be presented with whatever I was gonna cook up there.)
So it was time to try some jank...
And try some jank I did.

My first thought was `decimal`. C# has a built-in type called `decimal`, which still has a finite precision, but more of it than even `double`, and it attempts to approximate as what you'd call "Real Numbers" a little harder than floating point numbers do, by internally representing them as floating-point _base-10_ numbers.
In fact, it does that _so_ hard, that if you convert a `3.1f` literal float to a decimal, you'll get a perfect `3.1m` decimal back - even though the perfect `3.1f` float _never existed_ because that number is _unrepresentable_ in base-2 floating-point arithmetic, and what you _really_ had was `3.099999904632568359375`, and C# basically internally did things to assure you that you're dealing with the Spherical Cow and that floats aren't real.
(You'll probably ask, why doesn't everyone use these for everything all of the time, then? Same reason as always - they're slow.)

The language silently fixes things up by [essentially rounding the input number when performing the conversion from `float` to `decimal`, up to 7 decimal places of the mantissa](https://github.com/dotnet/runtime/blob/ce6cf4b2e2a8c7d80ed7b31487eaab574d9007fa/src/libraries/System.Private.CoreLib/src/System/Decimal.DecCalc.cs#L1497-L1659).
Any remaining garbage is basically discarded.
(This also happens for `double` to `decimal` conversions, too, by the way, but [the threshold there is 15 places](https://github.com/dotnet/runtime/blob/ce6cf4b2e2a8c7d80ed7b31487eaab574d9007fa/src/libraries/System.Private.CoreLib/src/System/Decimal.DecCalc.cs#L1661-L1831).)

Normally that _is_ what you want, but this was not a normal situation.
I *wanted* the garbage to persist because I was in the unreasonable position of attempting to approximate x87 register operations.
What now?

I'll tell you what now, but if you're a purist you might wanna avert your eyes.

One other fact about floating point numbers is that languages will try _very hard_ to lie to you that they are the Spherical Cow Real Numbers you want to think that they are.
But given enough confusion, they will throw their hands up and essentially tell you "alright, I tried, but now you're on your own".

As my luck had it, both implementations of the game had the method receive `float` parameters, not `double`.
Had it been `double`, I would have been a little hosed and would have had to basically write a manual conversion method that does not round - but because it was `float`, I could use a faster trick, namely *`float`-to-`double` promotion*.

When you cast a `float` to a `double`, the runtime does not attempt to guess what the number being cast is anymore.
It will essentially go, "alright, I'll move the bits around for you. but I'm not guaranteeing that it does what you want it to do".

To decipher that into human language: the runtime will take the sign, exponent, and mantissa bits from the `float`, and convert them just enough to adhere to the bit boundaries of `double`.
But - notably - this is *only* done via bit shifts or basic arithmetic.
The runtime does not care what the number is.
It does the same operations every time, namely:

- move the sign bit across,
- subtract the `float` bias from the exponent, add the `double` bias to the exponent, put that result where the exponent goes in a `double`,
- bit-shift the `float`'s mantissa a tad to the left to fill all of the available mantissa bits in a `double`.

The reason that this is significant is that if a decimal number isn't perfectly representable in base-2 floating-point arithmetic, this conversion can be _imprecise_ in a particular way.
What does that mean?
It means that it is possible (and in fact very likely) that for some number written in decimal, it holds that

```csharp
3.1d != (double)3.1f
```

Again, 3.1 does not accurately exist in base-2 computer land, so when converting from a decimal string to a floating-point number, the _compiler_ will look at the number string and then look at the type of the variable, and then emit the closest available floating-point number to the decimal string that it was given.
But that means that because `double`s are denser than the `float`s on the real number line, a `float`-to-`double` upcast will sometimes fail to produce that sort of closest estimate because it's lost the context that what was meant is a _decimal_ 3.1.
This is relevant because it allows me to fool C# into basically skipping the round-when-converting-to-decimal behaviour by doing:

```csharp
(decimal)(double)3.1f
```

This gives me:

- A number that is actually closer to the literally-interpreted number stored in the register, without computer white lies included.
  As in, the resulting number is closer to what the *actual* bits mean when strictly interpreted using the IEEE754 spec.
  It's the 3.099999 thing, warts and all, which you normally don't want, but _I do_, to make the following calculations more precise.
- A number of a higher precision than `double`, which means that the rounding operation at the end that was causing me trouble is more likely to produce the more accurate result that I want it to produce to match .NET Framework/x87.

Is it perfect?
By no means.
It is an ugly hack.
It's nowhere near guaranteed to be as precise.
But it works close enough (fixed all known cases of the discrepancy except one), and at the same time it is stupid enough to also be funny to write a blog post about it.

I do not like floating point numbers.
It feels I've lost too much of my life to them.
But I guess that they are the best thing that we have to work with, and at least I've dealt with them enough that I can better see that the cow is not _that_ spherical.

---
layout: post
category : Programming
tags : [c64, Bubble Bobble]
title: The Bubble Bobble Continue Bug
---

<style type="text/css">
	figure {
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
	figcaption {
		font-style: italic;
		text-align: center;
	}
	img {
		height: 100%;
		width: 100%;
		object-fit: contain;
		box-sizing: border-box;
        background: white;
	}
	pre {
		overflow-x: auto;
		overflow-y: hidden;
		padding-bottom: 20px;
		margin: 10px 0 10px 0;
	}
	ul {
		margin: 10px 0 15px 0;
	}

	.highlight {
		background-color: white !important;
	}
</style>

<img src="/assets/posts/2025-10-07-bubble-bobble-c64-continue-bug-fix/game-over.png">


The c64 version of Bubble Bobble has a credit system. When you lose all your lives you can press fire to use a credit and get 3 new lives. Just like if you dumped another coin in the arcade machine.

But that never worked right. If both players are out of lives, the game immediately exits to the home screen. The only workaround was to hold down the fire button while the death animation is playing, so the input was already active by the time the animation completed.


## The Shallan Approach

Some years ago, [Shallan created a fix](https://www.indieretronews.com/2019/09/bubble-bobble-c64-essential-continue.html) for this. He implemented a countdown, giving the player more time to react before getting kicked out of the game.

Shallan used the popular method of running the game in an emulator and dumping the memory right after the game loaded itself. Beware that the initial release was botched and will glitch out entirely if played far enough.


## The Remaster

The past 9 months, I've been working with the talented Davide Bottino on [Bubble Bobble C64 Remastered](https://daves-retro-forge.itch.io/bubble-bobble-c64-remastered). I built tools to replace almost all graphics in the game. They let Davide iterate on the art quickly, and helped him really polish all sprites and backgrounds.

<figure>
	<img src="/assets/posts/2025-10-07-bubble-bobble-c64-continue-bug-fix/remaster.jpg"/>
	<figcaption>Bubbles say "pon" when they pop. Very important.</figcaption>
</figure>

My toolchain works with the raw binary game prg-file directly, without a disassembler or compiler. Therefore, it is important to me that all memory regions stay in the same place so I can accurately read and patch them. For whatever reason, Shallan's version of the game has moved code and graphics around, making it unusable for me.


## My Take

I initially tried reimplementing Shallan's workaround in my version, but found it unsatisfying. It didn’t really address the root cause of the bug and, in my opinion, made dubious assumptions about unused memory regions. The whole approach felt off. I didn't really want to implement a countdown or any other new functionality, just fix the bug.

I dug around in the code some more and found the spot where the game checks for game over every frame:

```asm
lda player_1_status
ora player_2_status
beq game_over
```

* `lda player_1_status` loads the `player_1_status` to the `A` register.
* `ora player_2_status` bitwise OR:s the `A` register with `player_2_status`, saving the result to  the `A` register. 
* `beq game_over` jumps to the `game_over` section if the `Z` (zero) processor flag is cleared. In this context, that's if the result of the previous `ora` was zero.

Since the player statuses are zero when dead/not joined, getting a zero in the OR:ed together value means both are dead.

In C, I would have written:

```c
if (
	player_1_status == 0 &&
	player_2_status == 0
) {
	game_over();
}
```

I realized thinking of it in terms giving the player a few seconds to react misses the bigger picture. Why stress the player at all?

The purpose of an arcade machine is to swallow coins. But on a home computer, the player has all the time in the world. This arcade mindset was already misguided for home computers when the game was released in 1987, but it affected game design well into the 90s.

I can excuse the credit system, since it is after all an arcade port. But there is no reason at all to exit the game, unless all the credits are used up. That's a pretty simple fix though. You just need to include the credit count in the check:

```c
if (
	credits_left == 0 &&
	player_1_status == 0 &&
	player_2_status == 0
) {
	game_over();
}
```

Translating it back to asm is a bit more involved, because for some reason, Bubble Bobble stores the credits_left - 1 instead. There is an instruction to `inc`-rement by one, but that's not available for the `A` register, so I choose to load to the `Y` register instead and transfer to `A` after the increment:

```asm
ldy credits_left_minus_1
iny
tya
ora player_1_status
ora player_2_status
beq game_over
```

* `ldy credits_left_minus_1` loads the `credits_left_minus_1` value to the `Y` register.
* `iny` increments `Y`by one.
* `tya` transfers `Y` to `A`.

That means the `A` registry has the `credits_left` loaded, and both player statuses can be OR:ed with it one by one, just like the `player_2_status`was before.

What happens if the player *doesn't* press fire to continue? Nothing. Well, the enemies continue to run around aimlessly and the music plays. But the game happily waits indefinitely for a player to join or quit manually. Just like modern players would expect it to.

<figure>
	<img src="/assets/posts/2025-10-07-bubble-bobble-c64-continue-bug-fix/no-hurry.jpg"/>
	<figcaption>All the time in the world.</figcaption>
</figure>


## Just 4 More Bytes

If you prepend a couple of definitions, you can play around with the asm code on [https://www.masswerk.at/6502/assembler.html](https://www.masswerk.at/6502/assembler.html):

```asm
credits_left_minus_1 = $ab
player_1_status = $b2
player_2_status = $b3
game_over = $0a99
*= $0a64
```

The original code compiles to 6 bytes:

```
0a64  a5 b2
0a66  05 b3
0a68  f0 2f
```

My fix compiles to 10 bytes:

```
0a64  a4 ab
0a66  c8
0a67  98
0a68  05 b2
0a6a  05 b3
0a6c  f0 2b
```

I tried optimizing the surrounding code to squeeze in my fix, but nothing I tried helped.

I had just fixed an unrelated issue in the level initialization and knew there were five bytes to spare in that section. But that didn't help. Even just moving code to a separate function uses up at minimum 4 extra bytes, and possibly another 3 for existing code to jump past the new function, so the fix still wouldn't fit.

I read through the level initialization code again to see if I could optimize it just a tiny bit. This original code compiles to 55 bytes:

```asm
*=$09A5

; If first level.
bcc else
; Hide the level during scrolling.

lda #0
; Disable sprites
sta $d015
; A is set to 0/black.
jsr $e740 ; Set all color ram to A.

; Duplicated in the else-branch as well.
jsr $e000 ; Load next level to off-screen buffer.
jsr $37c9 ; Scroll in level from off-screen buffer.
jsr $e4c5 ; Write level number to screens and charset.

; Un-hide the level after scrolling.

lda #$0d ; Set A to green.
jsr $e740 ; Set all color ram to A.

jsr $7b53 ; Draw stats.

; Make the level number white.
lda #$09
sta $d800
sta $d801

jsr $e4da
lda #$ff
sta $d015 ; Enable sprites
jmp end_if

else:

; Duplicated in the if-branch as well.
jsr $e000 ; Load next level to off-screen buffer.
jsr $37c9 ; Scroll in level from off-screen buffer.
jsr $e4c5 ; Write level number to screens and charset.

end_if:
```

This code initializes each level. The first level gets special treatment since it is not supposed to scroll in. But instead of just skipping the scrolling animation, the game hides the screen while in level one. Weird choice. But look at how the screen is hidden:

```asm
lda #0
; Disable sprites
sta $d015
; A is set to 0/black.
jsr $e740 ; Set all color ram to A.
```

The sprites are hidden and the color ram is set to black. The the 2 extra background colors are already set to black elsewhere. Then, after the scrolling, the background colors are restored and the color ram is set to green. That's a whole lot of work just to hide the screen, *when a single register write is enough*. The color ram doesn't even need to be set to green. The software sprites that use the color ram all update it when drawn anyway.

There are also 3 function calls that are duplicated in both the if and the else branches. By refactoring them out of the if, the code can be shortened to just 32 bytes after compiling:

```asm
*=$09A5

; If first level.
bcc end_if
; Hide the level during scrolling.
lda #$0b
sta $d011 ; Screen control register
end_if:

jsr $e000 ; Load next level to off-screen buffer.
jsr $37c9 ; Scroll in level from off-screen buffer.
jsr $e4c5 ; Write level number to screens and charset.

; Un-hide the level regardless.
lda #$1b
sta $D011 ; Screen control register

jsr $7b53 ; Draw stats.

; Make the level number white.
lda #$09
sta $d800
sta $d801
```

My refactoring saved 23 bytes, which was more than enough for the continue bug fix.

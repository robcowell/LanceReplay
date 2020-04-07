# LanceReplay
## Lance's MOD Replayer for Atari STe with some fixes by RiFT


### "HOLY FUCK LADS I FIXED IT" - *Mrs Beanbag*

With our recent release, "Technology of the Ancients", we had a lot of chip-style MODs to play back on our beloved Atari.  To do them justice and make them sounds as "clean" as possible, we chose Lance's famous 50khz replay routine.  It's STe only as it uses the DMA for playback, but it's the highest sound quality replayer I'm aware of on the ST(e).

### BUT....

Chipmods are funny little beasts.  To keep the size down, they use tiny samples and use tons of Protracker commands and tricks on them to bend them to the musician's will.  We quickly discovered that the Lance replayer doesn't quite cover them all correctly, mostly in the area of the loop points on samples.

### You know that thing with the extending the loops? It's already doing it. But it's doing it wrong. - *Mrs Beanbag*

See this..

      mt_data   incbin big_numbers.mod`
                ds.w    31*640/2        ;These zeroes are necessary!`

that's why those zeroes are necessary, because that's where it extends the loops into

Now see this

        mtloop3 clr.l    (a2)
                move.l    a2,d1
                add.l    d2,d1
                move.l    d1,(a1)+
                moveq    #0,d1
                move.w    (a0),d1
                asl.l    #1,d1
                add.l    d1,a2
                add.w    #640,d2
                tst.w    4(a0)
                bne.s    .mt_no_test
                cmp.w    #1,6(a0)
                ble.s    .mt_no_test
                subq.w    #1,6(a0)
                move.w    #1,4(a0)

that `cmp #1,6(a0)` is where it tests if there's a loop on the sample, only the test is backwards. It should be followed by a `bgt`, not a `ble` - in fact really it should be `bhs` because unsigned comparison.

### Wait, there's more...
#### Some other things the amazing Mrs Beanbag found, largely documented below for historical reasons

* actually the biggie was the size of the data section that the mixer code got generated into, which wasn't quite big enough
* also just realised it's shifting all the sample data by one bit, which makes sense since it's mixing two 8 bit channels into 1 but it means the original samples are now essentially 7 bit - there's tricks to avoid having to do that but it means rewriting the mixer
* actually i just noticed some weird special-casing going on when the loop starts at the beginning of the sample, i don't understand the reasoning for it but it may be the difference between mods that work and mods that don't
* the code assumes that where there's a loop it extends to the end of the sample. which is normally reasonable otherwise it's a bit of a waste of space - when it resets the data pointer back to the start of the loop, it looks at how far over the end of the sample it's gone, but in this case it hasn't actually gone off the end of the sample it's only gone off the end of the loop
*  sound quality probably isn't perfect on account of no interpolation when resampling. nearest neighbour always sounds a bit crispy 


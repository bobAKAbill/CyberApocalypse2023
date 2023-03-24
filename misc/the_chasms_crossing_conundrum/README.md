# Cyber Apocalypse 2023

## The Chasms Crossing Conundrum

> As you and your trusty team of local pyramid experts stand at the precipice of the chasm, you catch a glimpse of the otherworldly relic glowing tantalizingly in the distance. But the final obstacle looms ahead - a narrow, unstable bridge that threatens to send you all tumbling into the depths below. It won't hold all of you at once. Time is running out, and the charge on your flashlight is dwindling. The chasm seems to be closing in, as if it's trying to swallow you whole. With each step, you feel the weight of the task at hand. The fate of your team, and perhaps even the world, rests on your shoulders. Can you summon the courage and skill to make it across in time, and claim the relic that lies just beyond your grasp?
>
>  README Author: [bobAKAbill](https://github.com/bobakabill)
>

Tags: _misc__

## Initial recon

Much like janken, upon connecting to this docker you're met with a prompt that *really* wants you to tell it something
```
1. Instructions
2. Play
> 
```

Let's see the instructions on how to play this game
```
☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ 
☠️                                                                             ☠️
☠️  [*] The path ahead is treacherous.                                         ☠️
☠️  [*] You have to find a viable strategy to get everyone across safely.      ☠️
☠️  [*] The bridge can hold a maximum of two persons.                          ☠️
☠️  [*] The chasm lurks on either side of the bridge waiting for those         ☠️
☠️      who think they can get across in total darkness.                       ☠️
☠️  [*] If two persons get across, one must come back with the flashlight.     ☠️
☠️  [*] The flashlight has energy only for a limited amount of time.           ☠️
☠️  [*] The time required for two persons to cross, is dictated by the slower. ☠️
☠️  [*] The answer must be given in crossing and returning pairs. For example, ☠️
☠️      [1,2],[2],... . This means that persons 1 and 2 cross and 2 gets back  ☠️
☠️       with the flashlight so others can cross.                              ☠️
☠️                                                                             ☠️
☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ ☠️ 
```
Sorry if that looks a little funky in your github view. This problem is a LOT like the one from your childhood about crossing a river with a farmer, a goat, a head of lettuce, and a fox. Except this time we only have x amount of time to do it in, and every person has a different amount of time they can cross in. We also know that the answer has to be given in the following format, `[x,y],[y],[y,z],[y]` Let's see what kind of input we have to play with.

```
It's right there! You can see it! You can almost touch it! 💠
Wait...
That bridge is not safe! It's too narrow and old and the chasm is too deep!
The equipment is heavy and if we all cross at once, the bridge will collapse, but you can't leave the relic behind.
Below are the estimates of how long each of us will take to cross the bridge and the charge left for the flashlight.

Person 1 will take 62 minutes to cross the bridge.
Person 2 will take 23 minutes to cross the bridge.
Person 3 will take 14 minutes to cross the bridge.
Person 4 will take 67 minutes to cross the bridge.
Person 5 will take 64 minutes to cross the bridge.
Person 6 will take 66 minutes to cross the bridge.
The flashlight has charge for 274 minutes. 🔦

Hurry now! We don't have much time!
Enter your strategy: 
```
Whelp, lets bust out the notepad

# One puzzle later

After a longer time than I'm willing to admit with a paper and pencil, I got the following move order
`[3,2],[3],[4,6],[2],[3,2],[3],[5,1],[2],[3,2]`
Trying it in the docker
```
Enter your strategy: 
> [3,2],[3],[4,6],[2],[3,2],[3],[5,1],[2],[3,2]

[-] Time limit exceeded! Too late!
```
Hrm... that's a bit odd. I know I got the right amount of time (do the math yourself if you want to). I'll try a few more times on my own real quick.

# Several puzzles later (the times change every time)
I dislike this problem. This problem keeps turning away my input. I'm going to go find some code online to do this problem (because I'm sure it's there somewhere)

Lo and behold, I find a perl script to do exactly what I want. It has some inputs for varying numbers of people and times (hardocded, unfortunately, but much faster than doing it myself).
```
#!/usr/bin/perl
#
# There is a dark night and there is a very old bridge above a
# canyon. The bridge is very weak and only 2 men can stand on it at
# the same time. Also they need an oil lamp to see holes in the bridge
# to avoid falling into the canyon.
# 
# Six man try to go through that bridge. They need 1,3,4,6,8,9(first
# man, second man etc.) minutes to pass the bridge.
# 
# What is the fastest way for those six men to pass this bridge?
#
# http://math.stackexchange.com/questions/792356/puzzle-about-six-travellers-going-through-bridge-above-canyon-with-an-oil-lamp

# node contains: left side, right side, lamp side, history, total time
my @q = ([["p1" .. "p8"], [], 0, [], 0]);
my %speed = (p1 => 7,p2 => 12,p3 => 33,p4 => 56,p5 => 97,p6 => 42, p7 => 19, p8 => 23);

my $min_speed = 10000;
while (@q) {
    my $node = pop @q;
    my $s = str($node);
    my $speed = speed($node);
    $best_time{$s} = $speed
        if !exists $best_time{$s} or $best_time{$s} > $speed;
    if (is_finished($node)) {
        if ($speed <= $min_speed) {
            print_node($node);
            $min_speed = $speed;
        }
    } else {
        my @new = next_pos($node);
        @new = grep speed($_) < $min_speed, @new;
        for my $node (@new) {
            my $s = str($node);
            push @q, $node if ! exists $best_time{$s}
                           or $best_time{$s} > speed($node);
        }
    }
}

sub is_finished {
    my ($node) = @_;
    @{$node->[0]} == 0;
}

sub speed { $_[0][4] }

sub print_node {
    my ($n) = @_;
    my (undef, undef, undef, $hist, $time) = @$n;
    print "Solution in $time units of time\n";
    print_hist($hist);
}

sub print_hist {
    my ($hist) = @_;
    return unless defined $hist;
    my ($last, $rest) = @$hist;
    print_hist($rest);
    print "  $last\n";
}

sub next_pos {
  my ($node) = @_;
  my ($lt, $rt, $lamp, $hist, $time) = @$node;
  my @s = ($lt, $rt);
  my @m = @{$s[$lamp]};   # men who can carry the lamp
  my @o = @{$s[1-$lamp]}; # men on the other side
  my @moves;
  for my $m1 (@m) {
    for my $m2 (@m) {
      next if $m2 le $m1;
      my @nma = remove(\@m, $m1, $m2);
      my @nmb = ( @o, $m1, $m2 );
      my @lr = $lamp == 0 ? ( \@nma, \@nmb ) : ( \@nmb, \@nma );
      my $d = $lamp == 0 ? "$m1,$m2 -> " : "<- $m1, $m2";
      my $sp = max(@speed{$m1,$m2});
      push @moves, [ @lr, 1-$lamp, [ $d, $hist ], $time+$sp ];
    }

    my @nma = remove(\@m, $m1);
    my @nmb = ( @o, $m1 );
    my @lr = $lamp == 0 ? ( \@nma, \@nmb ) : ( \@nmb, \@nma );
    my $d = $lamp == 0 ? "$m1 -> " : "<- $m1";
    push @moves, [ @lr, 1-$lamp, [ $d, $hist ], $time+$speed{$m1} ];
  }

  return @moves;
}

sub max {
    my $max = shift;
    $max = $max > $_ ? $max : $_ for @_;
    return $max;
}

sub str {
  my ($node) = @_;
  my ($lt, $rt, $lamp, $hist, $time) = @$node;
  return join "-", join("",sort @$lt), join("", sort @$rt), $lamp;
}

sub remove {
    my ($a, @rest) = @_;
    my %h = map { $_ => 1 } @$a;
    delete $h{$_} for @rest;
    return keys %h;
}
```

Let's try it!

# Three or four perls later
It still doesn't like my input. This is frustrating. The perl script outputs the move order, which can be typed by hand into the correct format, and also the time spent. I'm going to go ask the staff what's up with this

# After a support ticket
Turns out, there's actually a time limit for this whole process. You need to provide the docker with an answer within a certain amount of time, as well as having that answer fit within the time limits of the game. Personally, I have no clue how to refactor perl to output the answer in the right format, as well as take the time input as command line arguments. One thing I *do* know how to do is turn perl into Python (especially because I'm afraid I'm going to need to make my program connect to the docker itself, and I REALLY don't want to do that in perl). The perl can also take a while to run, because it does things recursively, and outputs the intermediate steps, and goes from the most ineffecient to the most effecient. I think the worst I saw was about 2 minutes of run time.

# My end result python
```
'''
def f(s):
    s.sort()                                   # sort input in place
    if len(s)>3:                               # loop until len(s) < 3
        a = s[0]+s[-1]+min(2*s[1],s[0]+s[-2])  # minimum time according to xnor paper
        return  a + f(s[:-2])                  # recursion on remaining people
    else:
        return sum(s[len(s)==2:])              # add last times when len(s) < 3

print(f([3, 1, 6, 8, 12, 32, 19, 23]))
'''

import sys
count = 0

people = ["1", "2", "3", "4", "5", "6", "7", "8"]
steps = ""

def two_left_case(a_side, b_side, indices):
    move_from_a_to_b(a_side[0], a_side[1], a_side, b_side, indices)


def three_left_case(a_side, b_side, indices):
    global steps
    move_from_a_to_b(max(a_side), min(a_side), a_side, b_side, indices)
    move_from_b_to_a(min(b_side), a_side, b_side, indices)
    print("Move %s back to A side" % str(min(b_side)))
    steps += "[" + people[indices.index(str(min(b_side)))] + "],"


def four_or_more_case(a_side, b_side, indices):
    move_from_a_to_b(min(a_side), sorted(a_side)[1], a_side, b_side, indices)
    move_from_b_to_a(min(b_side), a_side, b_side, indices)
    move_from_a_to_b(max(a_side), sorted(a_side)[-2], a_side, b_side, indices)
    move_from_b_to_a(min(b_side), a_side, b_side, indices)


def move_from_a_to_b(first_number, second_number, a_side, b_side, indices):
    global steps
    global count
    print(indices)
    print(type(indices))
    print(type(indices[0]))
    a_side.remove(first_number)
    a_side.remove(second_number)
    b_side.append(first_number)
    b_side.append(second_number)
    count += max(first_number, second_number)
    print("Move %s & %s to B side." % (str(first_number), str(second_number)))
    steps += "[" + people[indices.index(str(first_number))] + "," + people[indices.index(str(second_number))] + "],"


def move_from_b_to_a(number, a_side, b_side, indices):
    global steps
    global count
    b_side.remove(number)
    a_side.append(number)
    count += number
    print("Move %s back to A side." % str(number))
    steps += "[" + people[indices.index(str(number))] + "],"

if __name__ == '__main__':
    a_side = []
    n = len(sys.argv)
    for i in range(1,n):
        a_side.append(int(sys.argv[i]))
    indices = [str(x) for x in a_side]
    b_side = []
    while len(a_side) != 0:
        if len(a_side) == 1:
            print("Just bring that number to B...")
        if len(a_side) == 2:
            two_left_case(a_side, b_side, indices)
            print("Total step: %s" % str(count))
            break
        elif len(a_side) == 3:
            three_left_case(a_side, b_side, indices)
        else:
            four_or_more_case(a_side, b_side, indices)
    print(steps)
```
Okay, so I didn't ACTUALLY exactly refactor the perl code into python, but close enough (it makes it work).

I've got my little list of people at the top, so that way the steps I output are in people instead of [timeX,timeY] like my initial code did it. I also take advantage of pythons list.index() function to return the index of the first occurrence of an item in a list. This is also much more convenient because it takes times in command line arguments. Let's try this one (using the initial inputs I used as an example at the beginning)
```
└─# python3 pychasm.py 62 23 14 67 63 66                                                            1 ⨯
['62', '23', '14', '67', '63', '66']
<class 'list'>
<class 'str'>
Move 14 & 23 to B side.
Move 14 back to A side.
['62', '23', '14', '67', '63', '66']
<class 'list'>
<class 'str'>
Move 67 & 66 to B side.
Move 23 back to A side.
['62', '23', '14', '67', '63', '66']
<class 'list'>
<class 'str'>
Move 14 & 23 to B side.
Move 14 back to A side.
['62', '23', '14', '67', '63', '66']
<class 'list'>
<class 'str'>
Move 63 & 62 to B side.
Move 23 back to A side.
['62', '23', '14', '67', '63', '66']
<class 'list'>
<class 'str'>
Move 14 & 23 to B side.
Total step: 273
[3,2],[3],[4,6],[2],[3,2],[3],[5,1],[2],[3,2],
```
Awesome! I got the right format (ignore my prints, I had to deal with my list data not being the same type as other list data). The comma at the end is a little funky, but since for the first time around I'm just going to copy/paste by hand, it doesn't really matter. Now let's do it for real!
```
└─# nc -v 159.65.94.38 30942
159.65.94.38: inverse host lookup failed: Unknown host
(UNKNOWN) [159.65.94.38] 30942 (?) open

1. Instructions
2. Play
> 2

It's right there! You can see it! You can almost touch it! 💠
Wait...
That bridge is not safe! It's too narrow and old and the chasm is too deep!
The equipment is heavy and if we all cross at once, the bridge will collapse, but you can't leave the relic behind.
Below are the estimates of how long each of us will take to cross the bridge and the charge left for the flashlight.

Person 1 will take 77 minutes to cross the bridge.
Person 2 will take 70 minutes to cross the bridge.
Person 3 will take 97 minutes to cross the bridge.
Person 4 will take 73 minutes to cross the bridge.
Person 5 will take 63 minutes to cross the bridge.
Person 6 will take 39 minutes to cross the bridge.
Person 7 will take 32 minutes to cross the bridge.
Person 8 will take 85 minutes to cross the bridge.
The flashlight has charge for 613 minutes. 🔦

Hurry now! We don't have much time!
Enter your strategy: 
> [7,6],[7],[3,8],[6],[7,6],[7],[1,4],[6],[7,6],[7],[2,5],[6],[7,6]
You did it! You saved everyone! 🎉
Oh no, the relic vanished into thin air!
Approaching the point that the relic stood you see a note that reads:
HTB{4cro55_th3_br1dg3_4nd_th3_ch4sm_l13s_th3_tr34sur3}
```
Sweet! A flag! And I got to learn how to do the Midnight Train problem (or the bridge and torch problem, but who doesn't like a little bit of Journey)

## Flag
HTB{4cro55_th3_br1dg3_4nd_th3_ch4sm_l13s_th3_tr34sur3}
## UFO operator

This is a comparison operator that return -1 if the second operand is less than the first, zero if they are equal, or 1 if it’s greater than. A slightly niche function, though one that reminds me of an Opcode from a zachtronics puzzle game.
It does have it’s uses, especially when paired up with a match (a php8 feature):

```php
$competitionText = match ($yourScore <=> $TheirScore) {
     -1 => ‘is beating you’,
      0 => ‘you are tied’,
      1 => ‘you are winning’
};
```
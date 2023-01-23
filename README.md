# Unicode Hex Input fixed

MacOS's built-in Unicode Hex Input layout has a bug where all characters with 
a code both starting ending in 0 will not output anything. For example the
character `à` is code `00E0` and is impossible to type with the built in layout
(as of Jan. 2023).

Using this keylayout instead fixes the issue. 
It also fixes the missing text navigation shorcuts to move word by word using
the option+arrow key shortcuts.

## Installation

To install this layout:

- Download the keylayout file in the list above
- Copy or move it to either
    - `~/Library/Keyboard Layouts/`
    if you want the layout to be activated just for you user or 
    - `/Library/Keyboard Layouts/`
        if you want the keyboard to be available system-wide
- Log out and login again so macOS refreshes its list keyboard layouts

It should be now available to select in 
System Settings > Keyboard > Text input > Input sources > Edit...

## Further details

### The "starts and ends with 0" issue

I'm not sure what the bug exactly is with the built-in layout, but when creating
a copy of the built-in Unicode Hex Input using Ukulele, an empty string is found
in place of the null character `&#x0000;` which has a numeric value of 0. I 
suspect something similar happened with the built in layout when Apple must have
updated it at some point, stripping out the null character by mistake.

In order to be able to output all Unicode characters without storing all of
Unicode in the keyboard layout, this layout calculates the character to output.
That is done through "actions", which  are commonly used to create dead keys.
When holding the option key, each successive keypress on hexadecimal numbers
keys (0-9 plus ABCDEEF) will add that number to the state and multiply by 16,
with the fourth keypress adding the number from the state to a unicode character
of corresponding value and output this modified character. In the case of the
fourth keypress being 0, the corresponding character with value 0 is the null
character, which is missing. This means there is no character to add to and no
character to output.

Manually entering the null character's code where it belongs fixes the
issue.

#### Fixing the issue in a keylayout file generated with Ukulele

To generate your own copy of the Unicode Hex Input layout, select the built-in
Unicode Hex Input as your layout in macOS and generate a brand new keylayout
from the current input source in Ukulele. (File > New from current input source)

To find where the null character is missing, we can look for what happens when
pressing the 0 key while holding down the option key. In the `modifierMap`
section of the file, we need to find which `keyMap` gets activated by pressing
the option key alone. The `modifier` elements tell us what keys they're looking
for. `anyOption` means it doesn't matter whether it's the left or the right
option key, and `?` means holding that key doesn't matter. We want to find the
`modifier` element which has all the keys marked with a `?` except for option. 
That would mean that at least option needs to be pressed, and no other key is 
required. In my file, I have:
```xml
<keyMapSelect mapIndex="3">
    <modifier keys="anyShift? caps? anyOption"/>
</keyMapSelect>
```
This means the relevant `keyMap` has index `3` as indicated in the
`keyMapSelect` wrapping the `modifier` element. This index might be different in
your generated file.

So the `<keyMap index="3">` section of the file describes what happens when we
press keys while holding option on the keyboard. Now we want to know what
happens when pressing the 0 key. The 0 key on the keyboard has code 29, or 82 on
the numpad. In my particular file I can find:
```xml
<key code="29" action="0"/>
```
This means pressing 0 while holding option triggers the action with id `0`.

Towards the end of the file you will find where the actions are defined.
We're interested in the section starting with `<action id="0">`. This is where
you will find a `when` element which outputs an empty string. In place of the 
empty string, we insert `&#x0000;`, the null character, like so:
```diff
- <when state="1" through="256" output="" multiplier="16"/>
+ <when state="1" through="256" output="&#x0000;" multiplier="16"/>
```

Save the file, follow the installation steps above. That's it!

### The missing option + arrow word by word navigation

If you created your own copy of the built-in layout and followed the instructions
to fix the missing null character above, you may have noticed all of the `key`
elements trigger an action to calculate character codes, and none of them trigger
a straight output. The arrow keys (codes 123 to 126) aren't even present. When 
looking at the generated keylayout file for a different layout where the option
 + arrow shortcuts do work, in the `keyMap` corresponding to the option key,
we can find the arrow keys:
```xml
<key code="123" output="&#x001C;"/>
<key code="124" output="&#x001D;"/>
<key code="125" output="&#x001F;"/>
<key code="126" output="&#x001E;"/>
```
Simply add those lines to the appropriate `keyMap` section, save, follow the
installation steps above, and you should have a Unicode Hex Input that supports
option + arrow text navigation.

Credits and links
=================

Ukulele, the keyboard layout editor for macOS:
https://software.sil.org/ukelele/

The fix for the 0 bug was found in this Ukulele support discussion:
https://groups.google.com/g/ukelele-users/c/aUWW_QFLA6s/m/vhrIM08sBQAJ

The fix for the arrow keys was found on Apple Stackexchange here:
https://apple.stackexchange.com/questions/41657/use-option-arrows-with-unicode-hex-input/89970#89970

Documentation for the XML format of keylayout files:
https://developer.apple.com/library/archive/technotes/tn2056/_index.html#//apple_ref/doc/uid/DTS10003085

Keyboard key codes:
https://stackoverflow.com/questions/3202629/where-can-i-find-a-list-of-mac-virtual-key-codes

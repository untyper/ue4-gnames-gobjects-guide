# How to find Unreal Engine 4.1X GNames and GObjects

### Requirements

- C++ knowledge
- Unreal Engine 4.1X source files
- Cheat Engine

> // GObjects are arrays of FUObject???

GNames are arrays of `FNameEntry`

`FNameEntry` is a struct that looks like this in the source:

```c++
struct FNameEntry
{
private:
	/** Index of name in hash. */
	NAME_INDEX		Index;

public:
	/** Pointer to the next entry in this hash bin's linked list. */
	FNameEntry*		HashNext;

protected:
	/** Name, variable-sized - note that AllocateNameEntry only allocates memory as needed. */
	union
	{
		ANSICHAR	AnsiName[NAME_SIZE];
		WIDECHAR	WideName[NAME_SIZE];
	};
	// ... struct methods ...
}
```

Most GName arrays begin with FNameEntries: "None", "ByteProperty" etc, followed by the games unique FNameEntries.

For example "ByteProperty" FNameEntry would look like this in code:

```c++
struct FNameEntry {
	NAME_INDEX Index        = 78937; // random index
	FNameEntry* HashNext    = 0x77ff00; // pointer to next entry in linked list
	ANSICHAR AnsiName[1024] = "None";
}
```

### Finding GNames:

There's quite a few ways to find GNames but by far the easiest is the good old string search.
Since we know that most GNames start with "None" and are followed by "ByteProperty" etc, we can search for either one in Cheat Engine's string search.
You will most likely get multiple results. Maybe less than 10 maybe more than 1000. This obviously depends on what name you decide to search for and how large the game is.
In my experience searching for "ByteProperty" has never resulted in more than 300 results.
Usually GNames is the first result but there is no guarantee, so just click away until you find the correct one.

You know you have found the correct one when "None" is preceded by 16 bytes on 64-bit and nothing else (or question marks) AND followed by "ByteProperty", etc, etc

Grab the address of the first byte of the preceeding 16 bytes before "None". This is the begining of our GNames.

![Alt 1-png](https://raw.githubusercontent.com/untyper/ue4-gnames-gobjects-guide/main/img/1.png)

Our GNames should conform to the FNameEntry struct above, but it's a little bit difficult to visualise this in Cheat Engine's memory view so let's reorganize the bytes a little bit:

```
5E 60 2D 35                             // ??
F5 7F 00 08                             // Index
98 12 6F 0B 00 00 00 00                 // HashNext
4E 6F 6E 65 00                          // AnsiName "None"
00 00 00                                
--------------------------------------
5E 60 2D 35                             // ??
F5 7F 00 10                             // Index
08 FA FE 0F 00 00 00 00                 // HashNext
42 79 74 65 50 72 6F 70 65 72 74 79 00  // AnsiName "ByteProperty"
00 00 00
...
... etc.
```

Let's also dissect the data/structures (Memory View -> Tools -> Dissect data/structures)

![Alt 2-png](https://raw.githubusercontent.com/untyper/ue4-gnames-gobjects-guide/main/img/2.png)

It's the same pattern.
We can now confidently say that we have found the GNames address.

The next step is to figure out whether this address is static or not. If it is static, then we can get an offset to it by substracting the base address from it:
`GameBaseAddr - GNamesAddr = Offset to GNames from GameBaseAddr`
We can use this to get our GNames every time without having to worry about the address changing everytime we reopen the game... until the game gets updated that is :-) in which case you have to repeat the previous steps again.

If GNames is dynamic you won't be able to substract the GameBaseAddr from it without either getting a negative number or a wrong address.
Still, we have atleast two ways of getting something static to use in our code when our GNames location is dynamic, they are:

- Find a static pointer (or static multilevel pointer) that points to our GNames address --> Trickier to find but faster and more reliable in your application
- Signature scan using patterns of bytes --> Can be easier to implement but can be less reliable and slow down your application

> To be continued...

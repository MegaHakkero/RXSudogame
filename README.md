# RXSudogame
A shitty ncurses sudoku game + file format for UNIX systems

## Why the fuck did I write this?
I became really good at playing sudoku with my ages old Nokia brick phone (sudoku's actually the most entertaining game on it), so I started wanting to play it on the computer too. I didn't find any good enough implementations of it for terminals though, so I decided to write one myself. The issue I had with other implementations was a lack of any kind of leveling system, because on the implementation that my phone has there's 100 sudokus you can complete, which gives you a goal you can move toward. The code is horrible, but I like the result nonetheless.<br><br>

Yeah, I know, I have a pretty boring life.

## Compilation, dependencies etc.
I've used some POSIX features as well as some other UNIX-only features in my program, so this doesn't compile on Windows because it doesn't have any of said features.<br>
To compile this, I use
```sh
gcc main.c -Wall -std=gnu11 -ltinfo -lncurses -pthread -o rxsudogame
```
but `-std=gnu99` would probably work too. I just have a really new GCC.<br>
Note that even though there's no other compile-time dependencies, you need a program called [qqwing](https://qqwing.com/download.html "qqwing downloads") installed on your system to actually generate sudokus with my program, otherwise you'll just get an error while trying to do it. On Manjaro and Arch Linux, you can install it with
```sh
sudo pacman -S qqwing
```
and on Debian-based systems,
```sh
sudo apt-get install qqwing
```
does the trick. I think some BSDs have a package for it as well, you can try installing it with
```sh
sudo pkg install qqwing
```
I know, I know. qqwing has a library. I just didn't realise that before I already wrote the function for generating sudoku files. By the way, if you can't find qqwing in your package manager's repositories, you can just download the source code from that link up top and compile it. They have their own instructions for that.

## Instructions
You probably don't wanna read anymore half-useless jargon, so I'll just show you how to use the program.

### Generating and loading sudomaps
Now, you can't just start playing right after you've compiled this, you'll have to create a sudomap first (more about sudomaps in the `About that file format` section.<br>
To generate a sudomap, you can type
```sh
rxsudogame generate <count> <output name>
```
Just replace `<count>` with the number of games you want to include in the output file and replace `<output name>` with a file name of your choice. Just be sure not to type in an existing file's name, this program will overwrite it without prompting you. I was too lazy to add a check for that, lol<br>
So, now that you have a game generated, you can load it and start playing. Just type in
```sh
rxsudogame load <input file>
```
where `<input file>` is the file you generated with the previous command.<br><br>

If there are any errors while loading or generating a sudomap, the program will print them out and exit as per standard procedure. If generating a file succeeds, the program will exit silently. If loading a file succeeds, the program will tell you that, print the size of the games' pointers in memory and prompt you to press `<return>` to continue. After that, you can start playing.

### Playing the game
The UI layout is pretty easy to get, it's basically just a sudoku grid with a timer on top that tells you how many hours, minutes and seconds you've spent solving the current game. Below the grid, it tells you which game you're currently solving as well as whether you've solved it or not.<br><br>

The controls are also easy to get:
```md
- The number keys 0 through 9 modify the value in the current square. Keys 1 through 9 set a new value for the square and 0 clears an existing value. Note that you can't change a protected square's value. Those are indicated by a reverse color pattern.
- The WASD keys move the cursor around the grid, 'nuff said
- The left and right arrow keys save the current game you're solving and select another one. Left selects the previous game, right selects the next one.
- the Q key saves the current game and exits the program
- the C key checks whether the game is solved or not. If it is, it'll mark the game solved and save it, stopping the timer. If the game isn't solved, it'll highlight all incorrect squares with a reverse color pattern for 2 seconds.
```

## About that file format
You might've noticed that `+ file format` part in the beginning of this file, and if so, good - you can read. Basically, in order to have a leveled sudoku system, I had to create some kind of format to store many games in one file so that the player would have some way to manage their games and this was my solution; a simple binary format called `sudomap`. Below I've written down details about it's structure, so you can maybe write support for it in your own sudoku program? (I'd be honored!)

### Header
I don't really know if I should even call this a proper header, it's basically just two actual fields, one of which is a signature.
```md
1. Signature - char[6] - a null-terminated string that acts as a way for programs to verify the file format. This should always be equal to "RXDSF".
2. Count - uint16_t - the amount of sudokus in the file
3. Entry separator - uint8_t - separates the sudoku entries from the header. This should always be equal to 0xFF.
```

### Sudoku entry / sudogame
These are the actual things you'll be loading into memory in the program. Pretty straight-forward.
```md
1. Data - uint8_t[81] - the actual game data that the player should be able to modify.
2. Protectmap separator - uint8_t - separates the Data and Protectmap fields. Should always be equal to 0xFC
3. Protectmap - uint8_t[81] - this field tells you which numbers in the Data field are generated by the program and should be protected from the player. A "1" in an index of this field means that the same index in the Data field should be protected.
4. Solution separator - uint8_t - separates the Protectmap and Solution fields. Should always be equal to 0xFE
5. Solution - uint8_t[81] - the solution for the game in the Data field.
6. Data separator - uint8_t - separates the Solution field from the Time and Solve Status fields. Should always be equal to 0xFD
7. Time - uint32_t - how many seconds it has taken the player to solve the current game
8. Solve Status - uint8_t - whether the player has solved this game or not
9. Entry separator - uint8_t - separates sudoku entries from eachother. Should always be equal to 0xFF
```

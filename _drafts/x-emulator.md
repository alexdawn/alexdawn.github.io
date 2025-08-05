## A journey into emulators

Sometime ago I was working on the Nand to tetris project, If you are not aware of it
please go and check it out and the fantastic accompanying book. The project involves
building a 16-bit computer then creating an high level language for it as well as some system functions. It packs an
impressive amount of concepts into relatively simple exercises, though each chapter I've done so far I've always hit at least one rather sticky bit of debugging. Thankfully the course provides enough unit tests to fully verify you've got a working assembler, compiler.

The computer section wasn't too bad as I had previously made another very basic computer following some university slides to make another toy computer in a circuit simulation software called https://github.com/jarikomppa/atanua.

I had skipped part one of course designing the circuits and the CPU as I realised this was the same as a steam game I had played called MHRD (sadly this pretty much ends at when you create the CPU and you can't really create anything other than the specified CPU, for a more sandbox experience I'd recommend a game called Turing Complete which after you finish building the computer you then get given a number of zachlike programming puzzles)

Back to Hack/Jack there were some really cool programs made for it such as  2.5D castle wolfenstein style game and a screensaver style 3D doughnut render. The most mad project was some had built a full 3D raytracing engine in Hack.

Along the way I was somewhat unhappy with the supplied emulator and thought a interesting side
project would be to create my own:

https://github.com/alexdawn/nand-2-tetris/tree/main/project/emulator

For such as simple computer as Hack making an emulator for it was equally simple, the code is pretty much one big
switch statement to run different operators on two operands, with the RAM being an array.

The initial version in python was even slower than the official emulator. So I decided to try and refresh my rather stale C coding ability and rewrite the emulator in C, this was a surpassingly easy task, thankfully I had written the python emulator in a minimal typed functional style, since I had half a thought that it wouldn't do for more than a prototype, but not getting too invested in the language it was code easier to rewrite in another language. Initially I used the xxx bindings to be able to call the
C functions from python, this wasn't too bad and only need minimal definitions to work, this was great with a huge speed up compared to a pure python emulator, however I wasn't happy with the emulator being
a desktop application the next step was to get it onto the web but how? I looked into WASM and found it was quite easy to to build to
by using a docker container with the cross compilation tool. Instead of invoking with python I could now invoke the C function with
javascript and could use an HTML5 canvas to draw to, WASM had a respectable speed.

Sadly at this point I realised most of the coolest projects for HACK/JACK were written for the VM rather than the emulator and would compile to assembly too big to fit on 15-bit addressable ROM, even extending the HACK specification and considering the idea of adding
in some sort of paging system unfortunately seemed to make the emulator much slower and choppier, I'm guessing there is a practical limit to just how big you can make a single array before it starts to take a hit, instead it might be possible to have several different arrays of the existing size and just point to the active one? Sadly at this point my enthusiasm left not only would I then have to try and modify the assembler to handle paging adding complexly to the jump instructions to avoid overflows.
# Extracting OPN2 from ymfm

This repository extracts only the classes required to emulate the OPN2 (YM2612) chip from [ymfm](https://github.com/aaronsgiles/ymfm/tree/17decfae857b92ab55fbb30ade2287ace095a381), and refactors them into a single-header format with all necessary dependencies included.

Please note that the provided implementation, [`ymfm_opn2.hpp`](./ymfm_opn2.hpp), has been modified to remove all dependencies on the C++ STL.  
This change enables compatibility with bare-metal builds on the Raspberry Pi using the [Circle](https://github.com/rsta2/circle) framework.  
The STL removal and refactoring were automatically performed using OpenAI’s Codex (GPT-5).

---

## How to Use

See [`example.cpp`](./example.cpp) for a working example.

### 1. Include the header

In your source file (e.g., `example.cpp`), include [`vgmdrv.hpp`](./vgmdrv.hpp):

```c++
#include "vgmdrv.hpp"
```

### 2. Create an instance

Create an instance of the `VgmDriver` class, specifying the sampling rate (typically 44100 Hz) and number of channels (1 = Mono, 2 = Stereo):

```c++
const int sampling_rate = 44100;
const int channels = 1;
VgmDriver vgmdrv(sampling_rate, channels);
```

### 3. Load a `.vgm` file

Use the `VgmDriver::load` method to read a `.vgm` file:

```c++
if (!vgmdrv.load(buf.data(), buf.size())) {
    puts("Load failed!");
    return -2;
}
```

### 4. Render the samples

Use the `VgmDriver::render` method to continuously obtain sample data in chunks of 4096 samples until the end of the track or a loop is detected:

```c++
std::vector<int16_t> wav;
while (vgmdrv.getLoopCount() == 0 && !vgmdrv.isEnded()) {
    int16_t work[4096];
    vgmdrv.render(work, sizeof(work) / 2);
    wav.reserve(wav.size() + sizeof(work) / 2);
    wav.insert(wav.end(), std::begin(work), std::end(work));
}
printf("%zu samples generated.\n", wav.size());
```

> The example writes the rendered samples to a `.wav` file to avoid OS-specific audio dependencies.  
> In a typical game or SDK, you would instead feed the rendered data to an audio API such as SDL2 or DirectSound.

---

## How to Test

You can build and run the example with the following commands:

```bash
git clone https://github.com/suzukiplan/ymfm_opn2
cd ymfm_opn2
make
```

When executed, the program reads `test.vgm` included in this repository and outputs a `.wav` file:

```text
g++ -O2 -o example example.cpp
./example test.vgm test.wav
Detected YM2612: clocks=7670454Hz <supported>
3481600 samples generated.
Writing a wav file...
```

You can play the generated `test.wav` using a command such as `afplay` to verify that it matches the music heard in the following video:

https://www.youtube.com/watch?v=1FBO5BiD1Fg

> The video shows [`test.fur`](./test.fur) played in [Furnace Tracker](https://github.com/tildearrow/furnace).

---

## Integrating into Your Project

Simply add [`vgmdrv.hpp`](./vgmdrv.hpp) and [`ymfm_opn2.hpp`](./ymfm_opn2.hpp) to your project and include them as:

```c++
#include "vgmdrv.hpp"
```

Since the implementation is entirely header-only, integration should be straightforward.

---

## Limitations

- The VGM command set implemented in `vgmdrv` is minimal.  
  If your `.vgm` file produces an error like  
  `"Detected an unknown VGM command: xx"`,  
  you’ll need to add support for the missing command.
- Although not yet verified, the DAC for channel 6 is likely unsupported.

---

## License

[BSD 3-Clause License](./LICENSE)

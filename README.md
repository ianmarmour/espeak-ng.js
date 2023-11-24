# espeak-ng

eSpeak-NG speech synthesizer, compiled to JavaScript via Emscripten. This particular distribution is stripped of quite a few options
including support for mbroala, sonic, klatt and the speech-player itself. I'm not sure about the viability of including these options
and still compiling to WASM/JS.

## Usage

```ts
import ESpeakNg from "espeak-ng";

// Create an instance of espeak CLI with provided arguments.
const espeak = await ESpeakNg({
  arguments: [
    "--phonout",
    "generated",
    '--sep=""',
    "-q",
    "-b=1",
    `--ipa=3`,
    "-v",
    "en-us",
    '"Some text segement"',
  ],
});

// ESpeakNg is compiled with FS support so you can read it's output.
const phenoms = espeak.FS.readFile("generated", { encoding: "utf8" });
```

## Building

### Requirements

#### System Libraries

```
sudo apt-get install make autoconf automake libtool pkg-config
sudo apt-get install gcc
sudo apt-get install libsonic-dev
sudo apt-get install ronn
sudo apt-get install kramdown
sudo apt-get install libpcaudio-dev
```

#### Latest Emscripten

```
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
```

### Compilation

Compliation has a few issues mostly around some broken files in the espeak-ng repository please see the one correction
that is needed in the below shell script.

```bash
git clone https://github.com/espeak-ng/espeak-ng.git
cd espeak-ng
./autogen.sh
./configure --without-async --without-mbrola --without-sonic --without-pcaudiolib --without-klatt --without-speechplayer
make
cd src/ucd-tools/
# NOTE: You will need to modify modify the broken autogen.sh script to use CHANGELOG.md vs ChangeLog.md
./autogen.sh
./configure
make clean
emconfigure ./configure
emmake make clean
emmake make
cd ../..
emconfigure ./configure --without-async --without-mbrola --without-sonic --without-pcaudiolib --without-klatt --without-speechplayer
emmake make clean
emmake make src/espeak-ng
mv src/espeak-ng src/espeak-ng.bak
emcc -s EXPORT_ES6=1 -s MODULARIZE=1 -s 'EXPORT_NAME="ESpeakNG"' src/.libs/libespeak-ng.so src/espeak-ng.o -o src/espeak-ng.js --embed-file espeak-ng-data@/usr/local/share/espeak-ng-data/ -s TOTAL_MEMORY=32MB
```

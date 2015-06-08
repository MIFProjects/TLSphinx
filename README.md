# TLSphinx

TLSphinx is a Swift wrapper around [Pocketsphinx], a portable library based on [CMU Sphinx], that allow an application to perform speech recognition **withouth the audio leave the device**

This repository has two main parts. First is syntetized version of the [pocketsphinx](http://sourceforge.net/projects/cmusphinx/files/pocketsphinx/5prealpha/) and [sphinx base] repositories with a module map to access the library as a [Clang module]. This module is accessed under the name `Shpinx` and has two submodules: `Pocket` and `Base` in reference to _pocketsphinx_ and _sphinx base_.

The second part is `TLSphinx`, a Swift framework that use the `Sphinx` Clang module and expose a Swift-like API that talks to _pocketsphinx_.

## Usage

The framework provide three classes:
- `Config` describe the configuration needed to recognize the speech.
- `Decoder` is the main class that has the API to perform the decode.
- `Hypotesis` is the result of a decode attempt. It has a `text` and a `score` properties.

#### Config

Represents the _cmd_ln_t_ opaque structure in `Sphinx`. The default constructor take an array of tuples with the form `(param name, param value)` where _"param name"_ is the name of one of the parameters recognized for `Sphinx`. In this example we are passing the acustic model, the languaje model and the dictionary. For a complete list of recognized parameters check the [Sphinx docs].

The class has a public property to turn on-off the debug info from printed out from `Sphinx`:
```swift
public var showDebugInfo: Bool
```

#### Decoder

Represent the _ps_decoder_t_ opaque struct in `Sphinx`. The default constructor take a `Config` object as parameter.

This has the functions to perform the decode from a file or from the mic. The result is returned in an optional `Hypotesis` object, following the naming convention of the _Pocketsphinx_ API. The functions are:

To decode speech from a file:
```swift
public func decodeSpeechAtPath (filePath: String, complete: (Hypotesis?) -> ())
```
The audio pointed by `filePath` must have the following characteristics:
- single-channel (monaural)
- little-endian
- unheadered
- 16-bit signed
- PCM
- sampled at 16000 Hz

To control the size of the buffer used to read the file the `Decoder` class has a public property
```swift
public var bufferSize: Int
```

To decode a live audio stream from the mic:
```swift
public func startDecodingSpeech (utteranceComplete: (Hypotesis?) -> ())
public func stopDecodingSpeech ()
```

You can use the same `Decoder` instance many times.

#### Hypotesis

This struct represent the result of a _decode_ attempt. It has a `text` property with the best scored text and a `score` with the score value. This struct implement `Printable` so you can print it with `println(hypotesis_value)`.

### Examples

#### Process an Audio File

As an example let's see how to decode the speech in an audio file. To do so you first need to create a `Config` object and pass it to the `Decoder` constructor. With the decoder you can perform automatic speech recognition from an audio file like this:

```swift
import TLSphinx

let hmm = ...   // Path to the acustic model
let lm = ...    // Path to the languaje model
let dict = ...  // Path to the languaje dictionary

if let config = Config(args: ("-hmm", hmm), ("-lm", lm), ("-dict", dict)) {
  if let decoder = Decoder(config:config) {
      
      let audioFile = ... // Path to an audio file
      
      decoder.decodeSpeechAtPath(audioFile) {
          
          if let hyp: Hypotesis = $0 {
              // Print the decoder text and score
              println("Text: \(hyp.text) - Score: \(hyp.score)")
          } else {
              // Can't decode any speech because an error
          }
      }
  } else {
      // Handle Decoder() fail
  }
} else {
  // Handle Config() fail  
}
```
The decode is performed with the `decodeSpeechAtPath` function in the bacground. Once the process finish the `complete` closure is called in the main thread.

#### Speech from the Mic

```swift
import TLSphinx

let hmm = ...   // Path to the acustic model
let lm = ...    // Path to the languaje model
let dict = ...  // Path to the languaje dictionary

if let config = Config(args: ("-hmm", hmm), ("-lm", lm), ("-dict", dict)) {
  if let decoder = Decoder(config:config) {
      
      decoder.startDecodingSpeech {
          
          if let hyp: Hypotesis = $0 {
              println(hyp)
          } else {
              // Can't decode any speech because an error
          }
      }
  } else {
      // Handle Decoder() fail
  }
} else {
  // Handle Config() fail  
}

//At some point in the future stop listen to the mic
decoder.stopDecodingSpeech()

```

## Installation

We are still evaluating what path to follow in order to improve the integraration so in the mean time you should download the project from this repository and drag the _TLSpinx_ project to your XCode project. If you hit errors about missing headers and/or libraries for _Sphinx_ please add the `Spinx/include` to your header search path and `Sphinx/lib` to the library search path and mark it as `recursive`

## Author

BrunoBerisso, bruno@tryolabs.com

## License

TLSphinx is available under the MIT license. See the LICENSE file for more info.

[CMU Sphinx]: http://cmusphinx.sourceforge.net/
[Pocketsphinx]: http://cmusphinx.sourceforge.net/wiki/tutorialpocketsphinx
[sphinx base]: http://sourceforge.net/projects/cmusphinx/files/sphinxbase/5prealpha/
[Clang module]: http://clang.llvm.org/docs/Modules.html
[Sphinx docs]: http://cmusphinx.sourceforge.net/wiki/

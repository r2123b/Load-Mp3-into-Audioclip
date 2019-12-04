# Load Mp3 into Audioclip

## SYSTEM REQUIREMENT
 - `Unity 5.6.4` or above
 - `.Net 3.5` or above

## PREMISE
In a populer way, you can load .mp3 file by `UnityEngine.WWW class`. 
```cs
WWW www = new WWW("MyMp3File.mp3");
yield return www;
audiosource.clip = www.GetAudioClip();
```
However, you will get an error on both Mac or Windows operation system ...
```
Streaming of 'mp3' on this platform is not supported
UnityEngine.WWWAudioExtensions:GetAudioClip(WWW)
<Start>c__Iterator0:MoveNext() (at Assets/TestLoader.cs:16)
UnityEngine.SetupCoroutine:InvokeMoveNext(IEnumerator, IntPtr)
```

## SOLUTION
Therefore, we replace [UnityEngine.WWW](https://docs.unity3d.com/ScriptReference/WWW.html) with [NLayer](https://github.com/naudio/NLayer) that is `cross-platform` (Windows, Mac OS, iOS, and Android).

## CODE EXAMPLE

```cs
using System;
using NLayer;
using UnityEngine;

public static class Mp3Loader {
  private static MpegFile mpegFile = null;
  private static string _filePath = String.Empty;

  public static AudioClip LoadMp3(string filePath) {
    _filePath = filePath;

    mpegFile = new MpegFile(filePath);

    // assign mpegFile's info into AudioClip
    AudioClip ac = AudioClip.Create(System.IO.Path.GetFileNameWithoutExtension(filePath),
                                    (int)(mpegFile.Length / sizeof(float) / mpegFile.Channels),
                                    mpegFile.Channels,
                                    mpegFile.SampleRate,
                                    true,
                                    OnMp3Read,
                                    OnClipPositionSet);

    return ac;
  }

  // PCMReaderCallback will called each time AudioClip reads data.
  private static void OnMp3Read(float[] data) {
    int actualReadCount = mpegFile.ReadSamples(data, 0, data.Length);
  }

  // PCMSetPositionCallback will called when first loading this audioclip
  private static void OnClipPositionSet(int position) {
    mpegFile = new MpegFile(_filePath);
  }
}
```

We can simplify the previous code by using [in-line function](https://stackoverflow.com/questions/4900069/how-to-make-inline-functions-in-c-sharp) instead...
```cs
using NLayer;
using UnityEngine;

public static class Mp3Loader {
  public static AudioClip LoadMp3(string filePath) {
    string filename = System.IO.Path.GetFileNameWithoutExtension(filePath);

    MpegFile mpegFile = new MpegFile(filePath);

    // assign samples into AudioClip
    AudioClip ac = AudioClip.Create(filename,
                                    (int)(mpegFile.Length / sizeof(float) / mpegFile.Channels),
                                    mpegFile.Channels,
                                    mpegFile.SampleRate,
                                    true,
                                    data => { int actualReadCount = mpegFile.ReadSamples(data, 0, data.Length); },
                                    position => { mpegFile = new MpegFile(filePath); });

    return ac;
  }
}
```

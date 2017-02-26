## Performance



FsFIX performance figures below cover writing FIX messages to and reading them from byte arrays. The FIX messages that are written, and the byte arrays that are read from, are created outside of the scope of the timed tests. The timings were generated using [BenchmarkDotNet](http://benchmarkdotnet.org/), on a 2013 macbook pro with 16GB ram running Windows 10 (from bootcamp, not a VM).


```
// * Summary *

BenchmarkDotNet=v0.10.1, OS=Microsoft Windows NT 6.2.9200.0
Processor=Intel(R) Core(TM) i7-4960HQ CPU 2.60GHz, ProcessorCount=8
Frequency=2533206 Hz, Resolution=394.7567 ns, Timer=TSC
  [Host]    : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0
  RyuJitX64 : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0

Job=RyuJitX64  Jit=RyuJit  Platform=X64  

                                 Method |       Mean |    StdDev |  Gen 0 | Allocated |
--------------------------------------- |----------- |---------- |------- |---------- |
                 WriteMarketDataRequest |  3.1632 us | 0.0188 us | 0.5503 |   4.58 kB |
    WriteQuoteStatusRequestMsgWithGroup |  3.8108 us | 0.0119 us | 0.9308 |   7.02 kB |
 WriteQuoteStatusRequestWithNestedGroup |  3.8592 us | 0.0098 us | 0.9511 |   7.02 kB |
        WriteComplexNewOrderMultiLegMsg | 31.5503 us | 0.1591 us | 2.2705 |   22.8 kB |
                  ReadMarketDataRequest |  4.4830 us | 0.0099 us | 0.5198 |   4.34 kB |
        ReadQuoteStatusRequestWithGroup |  6.3434 us | 0.0174 us | 1.0091 |   8.56 kB |
 ReadQuoteStatusRequestWithNestedGroups |  8.5575 us | 0.0173 us | 1.3000 |  10.29 kB |
                ReadNewOrderMultilegMsg | 44.8649 us | 0.0687 us | 1.9287 |  21.12 kB |

*** Hints ***
Outliers
  BenchmarkMsgReadWrite.WriteQuoteStatusRequestMsgWithGroup: RyuJitX64    -> 1 outlier  was  removed
  BenchmarkMsgReadWrite.WriteQuoteStatusRequestWithNestedGroup: RyuJitX64 -> 2 outliers were removed
  BenchmarkMsgReadWrite.ReadQuoteStatusRequestWithGroup: RyuJitX64        -> 2 outliers were removed
  BenchmarkMsgReadWrite.ReadQuoteStatusRequestWithNestedGroups: RyuJitX64 -> 1 outlier  was  removed

// * Diagnostic Output - MemoryDiagnoser *
Note: the Gen 0/1/2 Measurements are per 1k Operations
```


FsFIX is not fast enough for high frequency trading, but should be suitable for other applications. I will put some time into improving performance at some stage.


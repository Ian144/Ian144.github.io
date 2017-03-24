# FsFIX Performance Compared to QuickFixN and QuickFixJ

FsFIX performance is compared with QuickFixN (C#) and QuickFixJ (Java). Performance is in terms of the time taken to write and read FIX messages to and from byte arrays. [BenchmarkDotNet](http://benchmarkdotnet.org/) was used to run the FsFIX and QuickFixN benchmarks. [JMH](http://openjdk.java.net/projects/code-tools/jmh/) was used for QuickFixJ. The GitHub repo for QuickFixN and QuickFixN benchmark projects can be found [here](https://github.com/Ian144/FsFIX_benchmarks). The times are the average (mean) times in microseconds.

([the main FsFIX page is here](readme.md))

## Results

|                                  benchmark  | FsFIX<sup>1</sup>     | FsFIX (optimized)     |  QuickFixN (C#)  | QuickFixJ (Java)
| ------------------------------------------- |----------- |---------------------- |----------------- |-----------------
| write MarketDataRequest                     |  3.1550 us | 2.5762 us             |   8.4914 us      | 1.459 us
| write QuoteStatusRequest with group         |  3.7849 us | 3.3569 us             |  10.6389 us      | 1.216 us
| write QuoteStatusRequest with nested group  |  5.3959 us | 4.5921 us             |  19.7647 us      | 3.094 us
| write NewOrderMultiLeg                      | 30.2887 us | 20.0561 us            |  43.2407 us      | 5.880 us
| read MarketDataRequest                      |  4.4538 us | 2.7945 us             |  12.2478 us      | 1.956 us
| read QuoteStatusRequest with group          |  6.3069 us | 4.9191 us             |  11.9883 us      | 1.910 us
| read QuoteStatusRequest with nested groups  |  8.3278 us | 6.3515 us             |  19.5073 us      | 3.596 us
| read NewOrderMultileg                       | 44.1178 us | 31.0434 us            | 107.2047 us      | 14.456 us

FsFIX is faster than QuickFIXN and slower than QuickFixJ. It is surprising that QuickFIXN is some much slower than QuickFixJ, they seem to have a similar implementation. QuickFixNs performance is probably not an issue in the context in which it is used.

These figures can be compared to [CoralFIX](http://www.coralblocks.com/index.php/2014/07/coralfix-performance-numbers/), a commercial high performance FIX engine. CoralFIX benchmarks ran on a much higher performance PC than my laptop, but it is still interesting to compare.

<sup>1</sup> re-ran the benchmark for the original unoptimized version of FsFIX, as the original "WriteQuoteStatusRequestWithNestedGroup" benchmark had used a non nested-group FIX message.


## FsFIX optimizations

- FsFIX indexes byte buffers containing a FIX message. This index now optimistically assumes fields are in the order specified in the FIX specification, so the next field to read is one after the previous. 
- Improved conversion of primitive types when writing to byte arrays, ToString() is faster than sprintf
- non-allocating BytesToInt32 and BytesToUint32 conversions

There is still scope for making FsFIX faster, possibly as fast as QuickFixJ, but it is hard to be sure.

## How to run the benchmarks

Build the projects, in release mode for FsFIX and QuickFixN. Shutdown all other running programs.

FsFIX
1. open console at: <path to FsFIX>\benchmark\bin\Release
2. run: benchmark.exe

QuickFixN
1.    open console at: <path to your FsFIX_benchmarks checkout>\benchmarkQFDotNet\benchmarkQFDotNet\bin\Release
2.    run: benchmarkQFDotNet.exe

QuickFixJ 
1.    open console at: <path to your FsFIX_benchmarks checkout>\benchmarkQFJava\target
2.    run: java -jar target\benchmark.jar



## Summary

FsFIX is already faster than QuickFIXN before optimization, while having the benefit of better type checking. It may be possible to close the gap with QuickFixJ.


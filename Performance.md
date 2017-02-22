## Performance



// * Summary *

BenchmarkDotNet=v0.10.1, OS=Microsoft Windows NT 6.2.9200.0
Processor=Intel(R) Core(TM) i7-4960HQ CPU 2.60GHz, ProcessorCount=8
Frequency=2533210 Hz, Resolution=394.7561 ns, Timer=TSC
  [Host]    : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0
  RyuJitX64 : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0

Job=RyuJitX64  Jit=RyuJit  Platform=X64

                   Method |       Mean |    StdDev |  Gen 0 | Allocated |
------------------------- |----------- |---------- |------- |---------- |
 WriteNewOrderMultilegMsg | 30.9455 us | 0.2135 us | 2.1403 |  22.77 kB |
  ReadNewOrderMultilegMsg | 46.2663 us | 0.3090 us | 2.0589 |  21.12 kB |
            WriteLogonMsg |  3.3772 us | 0.0187 us | 0.4425 |   3.96 kB |




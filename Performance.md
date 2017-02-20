## Performance








// * Detailed results *
BenchmarkNewOrderMultilegMsgRead.ReadIncHeaderTrailer_FIXBufIndexIsAParam: RyuJitX64(Jit=RyuJit, Platform=X64)
Runtime = Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0; GC = Concurrent Workstation
Mean = 46.3376 us, StdErr = 0.0185 us (0.04%); N = 13, StdDev = 0.0667 us
Min = 46.2594 us, Q1 = 46.2831 us, Median = 46.3221 us, Q3 = 46.3726 us, Max = 46.5070 us
IQR = 0.0895 us, LowerFence = 46.1489 us, UpperFence = 46.5068 us
ConfidenceInterval = [46.3014 us; 46.3739 us] (CI 95%)
Skewness = 1.03, Kurtosis = 3.54


BenchmarkNewOrderMultilegMsgRead.Read: RyuJitX64(Jit=RyuJit, Platform=X64)
Runtime = Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0; GC = Concurrent Workstation
Mean = 42.6460 us, StdErr = 0.0228 us (0.05%); N = 12, StdDev = 0.0789 us
Min = 42.5118 us, Q1 = 42.5991 us, Median = 42.6398 us, Q3 = 42.6729 us, Max = 42.8058 us
IQR = 0.0738 us, LowerFence = 42.4885 us, UpperFence = 42.7835 us
ConfidenceInterval = [42.6013 us; 42.6907 us] (CI 95%)
Skewness = 0.43, Kurtosis = 2.51


Total time: 00:00:37 (37.46 sec)

// * Summary *

BenchmarkDotNet=v0.10.1, OS=Microsoft Windows NT 6.2.9200.0
Processor=Intel(R) Core(TM) i7-4960HQ CPU 2.60GHz, ProcessorCount=8
Frequency=2533211 Hz, Resolution=394.7559 ns, Timer=TSC
  [Host]    : Clr 4.0.30319.42000, 32bit LegacyJIT-v4.6.1586.0
  RyuJitX64 : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0

Job=RyuJitX64  Jit=RyuJit  Platform=X64  

                                   Method |       Mean |    StdDev |  Gen 0 | Allocated |
----------------------------------------- |----------- |---------- |------- |---------- |
 ReadIncHeaderTrailer_FIXBufIndexIsAParam | 46.3376 us | 0.0667 us | 2.0915 |  21.12 kB |
                                     Read | 42.6460 us | 0.0789 us | 1.9694 |  19.99 kB |

*** Hints ***
Outliers
  BenchmarkNewOrderMultilegMsgRead.ReadIncHeaderTrailer_FIXBufIndexIsAParam: RyuJitX64 -> 2 outliers were removed
  BenchmarkNewOrderMultilegMsgRead.Read: RyuJitX64                                     -> 3 outliers were removed

// * Diagnostic Output - MemoryDiagnoser *
Note: the Gen 0/1/2 Measurements are per 1k Operations



// * Detailed results *
BenchmarkNewOrderMultilegMsgWrite.WriteIncHdrTrlr: RyuJitX64(Jit=RyuJit, Platform=X64)
Runtime = Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0; GC = Concurrent Workstation
Mean = 31.2878 us, StdErr = 0.0386 us (0.12%); N = 13, StdDev = 0.1392 us
Min = 30.9724 us, Q1 = 31.2221 us, Median = 31.2847 us, Q3 = 31.3686 us, Max = 31.5653 us
IQR = 0.1465 us, LowerFence = 31.0024 us, UpperFence = 31.5883 us
ConfidenceInterval = [31.2122 us; 31.3634 us] (CI 95%)
Skewness = -0.22, Kurtosis = 3.39


BenchmarkNewOrderMultilegMsgWrite.Write: RyuJitX64(Jit=RyuJit, Platform=X64)
Runtime = Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0; GC = Concurrent Workstation
Mean = 27.0506 us, StdErr = 0.1837 us (0.68%); N = 15, StdDev = 0.7115 us
Min = 26.1032 us, Q1 = 26.4117 us, Median = 26.8949 us, Q3 = 27.6236 us, Max = 28.2554 us
IQR = 1.2120 us, LowerFence = 24.5937 us, UpperFence = 29.4416 us
ConfidenceInterval = [26.6905 us; 27.4107 us] (CI 95%)
Skewness = 0.33, Kurtosis = 1.54


Total time: 00:00:24 (24.39 sec)

// * Summary *

BenchmarkDotNet=v0.10.1, OS=Microsoft Windows NT 6.2.9200.0
Processor=Intel(R) Core(TM) i7-4960HQ CPU 2.60GHz, ProcessorCount=8
Frequency=2533211 Hz, Resolution=394.7559 ns, Timer=TSC
  [Host]    : Clr 4.0.30319.42000, 32bit LegacyJIT-v4.6.1586.0
  RyuJitX64 : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0

Job=RyuJitX64  Jit=RyuJit  Platform=X64  

          Method |       Mean |    StdDev |  Gen 0 | Allocated |
---------------- |----------- |---------- |------- |---------- |
 WriteIncHdrTrlr | 31.2878 us | 0.1392 us | 2.2705 |  22.77 kB |
           Write | 27.0506 us | 0.7115 us | 2.0915 |  20.86 kB |

*** Hints ***
Outliers
  BenchmarkNewOrderMultilegMsgWrite.WriteIncHdrTrlr: RyuJitX64 -> 2 outliers were removed

// * Detailed results *
BenchmarkWriteLogon.WriteLogonMsg: RyuJitX64(Jit=RyuJit, Platform=X64)
Runtime = Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0; GC = Concurrent Workstation
Mean = 1.4974 us, StdErr = 0.0010 us (0.07%); N = 12, StdDev = 0.0034 us
Min = 1.4920 us, Q1 = 1.4950 us, Median = 1.4978 us, Q3 = 1.4998 us, Max = 1.5033 us
IQR = 0.0049 us, LowerFence = 1.4877 us, UpperFence = 1.5071 us
ConfidenceInterval = [1.4954 us; 1.4993 us] (CI 95%)
Skewness = -0.04, Kurtosis = 1.82


BenchmarkWriteLogon.WriteLogonIncHdrTrl: RyuJitX64(Jit=RyuJit, Platform=X64)
Runtime = Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0; GC = Concurrent Workstation
Mean = 3.4694 us, StdErr = 0.0297 us (0.86%); N = 15, StdDev = 0.1151 us
Min = 3.3103 us, Q1 = 3.3801 us, Median = 3.4000 us, Q3 = 3.5790 us, Max = 3.6248 us
IQR = 0.1989 us, LowerFence = 3.0817 us, UpperFence = 3.8774 us
ConfidenceInterval = [3.4112 us; 3.5276 us] (CI 95%)
Skewness = 0.09, Kurtosis = 1.17


Total time: 00:00:30 (30.84 sec)

// * Summary *

BenchmarkDotNet=v0.10.1, OS=Microsoft Windows NT 6.2.9200.0
Processor=Intel(R) Core(TM) i7-4960HQ CPU 2.60GHz, ProcessorCount=8
Frequency=2533211 Hz, Resolution=394.7559 ns, Timer=TSC
  [Host]    : Clr 4.0.30319.42000, 32bit LegacyJIT-v4.6.1586.0
  RyuJitX64 : Clr 4.0.30319.42000, 64bit RyuJIT-v4.6.1586.0

Job=RyuJitX64  Jit=RyuJit  Platform=X64  

              Method |      Mean |    StdDev |  Gen 0 | Allocated |
-------------------- |---------- |---------- |------- |---------- |
       WriteLogonMsg | 1.4974 us | 0.0034 us | 0.2815 |   2.06 kB |
 WriteLogonIncHdrTrl | 3.4694 us | 0.1151 us | 0.4506 |   3.96 kB |

*** Hints ***
Outliers
  BenchmarkWriteLogon.WriteLogonMsg: RyuJitX64 -> 3 outliers were removed

// * Diagnostic Output - MemoryDiagnoser *
Note: the Gen 0/1/2 Measurements are per 1k Operations

// ***** BenchmarkRunner: End *****

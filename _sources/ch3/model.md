# Producer-Consumer Model

* In a typical DSP application scenario, a signal *source* (e.g., an ADC
  or a camera) provides a stream of samples that is fed into a DSP
  kernel (e.g., an FIR filter) for processing. The DSP kernel may
  produce an output stream of samples that is in turn fed into
  another DSP kernel (e.g., an FFT) for further processing, and so on,
  eventually reaching a signal *sink* (e.g., the PS host).

* This configuration naturally leads to the ***producer-consumer
  model*** in which a DSP kernel acts as a *producer* of samples and
  another DSP kernel downstreams acts as a *consumer* of the samples
  supplied by the producer kernel. 


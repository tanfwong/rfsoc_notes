# Producer-Consumer Model

* In a typical DSP application scenario, a signal *source* (e.g., an ADC
  or a camera) provides a stream of samples that is fed into a DSP
  task (e.g., an FIR filter) for processing. The DSP task may
  produce an output stream of samples that is in turn fed into
  another DSP task (e.g., an FFT) for further processing, and so on,
  eventually reaching a signal *sink* (e.g., the PS host). We may
  perform each DSP task in a separate PL kernel, or package all DSP
  tasks into a single PL kernel in HLS. 

* This configuration naturally leads to the ***producer-consumer
  model*** in which a DSP task acts as a *producer* of samples and
  another DSP task downstreams acts as a *consumer* of the samples
  supplied by the producer task. Similarly, we may also consider the
  same producer-consumer model at the finer level for operations with
  a DSP task.


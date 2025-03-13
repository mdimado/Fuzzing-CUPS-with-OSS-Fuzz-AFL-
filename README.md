# OpenPrinting Evaluation Task: Fuzzing CUPS with OSS-Fuzz and AFL++

This document tracks my progress in running `fuzz_ipp` with OSS-Fuzz and fuzzing CUPS with AFL++ as part of the OpenPrinting evaluation task.

## Task 1: Running `fuzz_ipp` in OSS-Fuzz

### Steps Followed:
- Cloned the OSS-Fuzz repository locally.
- Built the fuzzers for CUPS.
- Encountered error:
  - `manifest for gcr.io/oss-fuzz/cups:latest not found: manifest unknown: Failed to fetch "latest"`
- Checked Docker images, which showed that the image existed locally.
- Manually ran the container.
- Successfully entered the container, but `/out` was empty (fuzzers were not built properly).
- Rebuilt fuzzers with cleanup.
- Verified the output directory with `ls build/out/cups` to check for `fuzz_ipp`.
- Successfully built the fuzzers.
- Ran the fuzzer inside the container.
- Fuzzer executed successfully, showing output:
  - No crashes detected.

### Analyzing the Output:
- **New / Reduce**: Indicates a newly discovered codepath or optimizes the test cases.
- **cov**: Number of coverage points.
- **ft**: Number of unique execution paths discovered.
- **corp**: Number of test cases in the corpus.
- **exec/s**: Execution speed (per second).
- **rss**: Memory usage by the fuzzer.
- **Mutation Strategies:**
  - CopyPart
  - CrossOver
  - InsertByte
  - EraseBytes
  - PersAutoDict

## Task 2: Running AFL++ on CUPS

### Steps Followed:
- Set up AFL++.
- Encountered error:
  - Installation errors (`afl-clang-lto: command not found`).
  - Spent time debugging (clearing package cache, updating sources, retrying, updating `$PATH`).
- Cloned the CUPS repository.
- Identified CUPS components.
- Initially tried fuzzing the main CUPS daemon (`cupsd`).
  - `cupsd` is a system service, not a direct input-handling binary (not an ideal fuzzing target).
- To identify components to fuzz, searched for `ipp`.
  - Since `ipp` actively processes external inputs, it was a logical choice.
- Targeted the `ipp` parser (`ipptool`), which handles `ipp` requests.
  - Confirmed that it processes `ipp` requests.
- Validated the instrumentation by running:
  ```sh
  afl-showmap -o /dev/null -m none -- ./tools/ipptool -h
  ```
- Ran AFL++ fuzzing:
  ```sh
  afl-fuzz -i ~/cups_afl_corpus -o ~/cups_afl_results -- ./tools/ipptool -t @@
  ```

### Result:
- Displays number of cycles, corpus count, saved crashes, and saved hangs.
- Table shows `last new find: none yet` (odd, check syntax!).
- Possible reasons for `odd, check syntax!` message:
  - Input corpus isn't diverse enough (most likely).
  - Binary isn't properly instrumented.
- Shows the current mutation stage: `havoc` (aggressive fuzzing strategy).
- Low map density (0.38%) indicates AFL++ is covering only a small fraction of the instrumented code.

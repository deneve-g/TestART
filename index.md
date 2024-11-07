---
title: TestART
---

## Abstract

Unit testing is crucial for detecting bugs in individual program units but consumes time and effort. Recently, large language models (LLMs) have demonstrated remarkable capabilities in generating unit test cases.

However, several problems limit their ability to generate high-quality unit test cases:

1. Compilation and runtime errors caused by the hallucination of LLMs
2. Lack of testing and coverage feedback information restricting the increase of code coverage
3. The repetitive suppression problem causing invalid LLM-based repair and generation attempts

To address these limitations, we propose TestART, a novel unit test generation method. TestART improves LLM-based unit testing via co-evolution of automated generation and repair iteration, representing a significant advancement in automated unit test generation.

TestART leverages the template-based repair strategy to effectively fix bugs in LLM-generated test cases for the first time. Meanwhile, TestART extracts coverage information from successful test cases and uses it as coverage-guided testing feedback. It also incorporates positive prompt injection to prevent repetition suppression, thereby enhancing the sufficiency of the final test case.

This synergy between generation and repair elevates the correctness and sufficiency of the produced test cases significantly beyond previous methods. In comparative experiments:

- TestART demonstrates an 18% improvement in pass rate and a 20% enhancement in coverage across three types of datasets compared to baseline models
- It achieves better coverage rates than EvoSuite with only half the number of test cases

These results demonstrate TestART's superior ability to produce high-quality unit test cases by harnessing the power of LLMs while overcoming their inherent flaws.

## Approach

![TestART](./assets/img/TestART.jpg)

The workflow of TestART is illustrated as shown above. Given a source code, TestART initially performs pre-processing to alleviate the hallucination problem. Afterward, TestART leverages the ChatGPT-3.5 model to generate the initial set of test cases T<sub>candidate</sub>. 

Once generated, T<sub>candidate</sub> enters the co-evolution loop (depicted by the yellow arrows in the figure), which aims to produce high-quality final test cases. Specifically, TestART submits T<sub>candidate</sub> for compilation and execution to capture error messages.

Based on the captured errors (compilation errors, runtime errors, or other detected bugs), TestART applies the repair strategy to address these errors and recompiles and runs until it passes without bugs or reaches the maximum number of iterations. If the test cases still unsuccessful, T<sub>candidate</sub> is discarded. Otherwise, T<sub>candidate</sub> is promoted to T<sub>success</sub>. 

Next, TestART employs JUnit and OpenClover to evaluate the coverage of T<sub>success</sub> over the source code, transforming the uncovered areas into test feedback. If T<sub>success</sub> meets the coverage standard, it will be output as the final result (T<sub>final</sub>). Otherwise, T<sub>success</sub> and the coverage-guidance feedback are provided as positive prompt injections to the ChatGPT-3.5 model, continuing to the next co-evolution loop and revert to T<sub>candidate</sub>. 

TestART leverages the co-evolution mechanism between automated test generation and repair. Benefiting from this synergy, TestART can iterate test cases incrementally and ensure that each round of test cases passes successfully, which continuously improves coverage.

## Experiment Design

### Research Questions

**RQ1:** How does the correctness of the test case of TestART compare to the baseline? What are the error types in the test cases and how well does TestART repair different error types?

**RQ2:** How does the sufficiency of the test case of TestART compare to the baseline?

**RQ3:** How does the combination of different parts impact the robustness of TestART?

**RQ4:** What is the performance of TestART when applied to unlearned datasets, and how well does it generalize to new data?

### Datasets

To verify our method, we select three types of datasets:

1. **Defects4J**: A widely-used open-source dataset containing reproducible software bugs
2. **HITS dataset**: An unlearned dataset created after GPT-3.5's training cutoff date
3. **Internal dataset**: A proprietary dataset from a major tech company

For all datasets, we extract public, non-abstract classes as focal classes and their public methods as focal methods. The specific dataset configurations are shown in Table 1.

### Baselines

To evaluate the effectiveness of our proposal, we compare TestART with four baselines:

1. **EvoSuite**: A traditional Search-Based Software Testing (SBST) tool using evolutionary algorithms
2. **A3Test**: A deep learning-based test generation approach enhanced with assertion knowledge
3. **ChatGPT**: Two versions (ChatGPT-3.5 and ChatGPT-4.0) are used as baselines
4. **ChatUniTest**: A ChatGPT-based test generation tool using Generation-Validation-Repair framework

### Evaluation Metrics

We evaluate TestART across four key perspectives with the following metrics:

**Correctness**
- Syntax Error (SE): Percentage of test code with Java syntax errors
- Compile Error (CE): Percentage of test code with compilation errors  
- Runtime Error (RE): Percentage of test code with runtime errors/failures
- Pass Rate (Pass): Percentage of test code that compiles and runs successfully

**Sufficiency** 
- Branch Coverage of Correct Tests (BCCT): Branch coverage for passed focal methods
- Line Coverage of Correct Tests (LCCT): Line coverage for passed focal methods
- Total Branch Coverage (TBC): Overall branch coverage across all focal methods
- Total Line Coverage (TLC): Overall line coverage across all focal methods

**Error Detection**
- Mutation Coverage (MC): Ratio of killed mutations to total mutations
- Test Strength (TS): Ratio of killed mutations to covered mutations

**Test Case Count**
- Test Case Count (TCC): Total number of generated test cases

### Experiment Setup

In our experiments, TestART generates unit tests for each focal method through up to four iterations. The system selects the best test case by prioritizing execution success, maximum coverage, and minimal test count. 

For fair comparison with baselines:
- TestART and ChatUniTest use GPT-3.5-turbo-0125 as the base model (16k context length, temperature = 0.5)
- ChatUniTest's maxPromptTokens is set to 16,385 with default maxRounds of five iterations per attempt
- ChatGPT-3.5 and ChatGPT-4.0 baseline tests are obtained using TestART's initial generation
- A3Test is trained on the Methods2Test dataset with learning rate 1e-5 for 110 epochs
- EvoSuite is configured with:
  - 3 CPU cores
  - 2000MB memory per core
  - Maximum search time of 10 minutes

The testing environment uses:
- Java 1.8 as compiler and runtime
- JUnit 4 as testing framework
- OpenClover for coverage calculation
- PITest for mutation coverage (with 11 default mutators)

All experiments are repeated three times with averaged results.

## Conclusion

In this work, we introduce TestART, the first approach to integrate the traditional automated repair technique with the generative capabilities of LLMs through an innovative co-evolutionary framework for generating high-quality unit test cases.

TestART also introduces positive prompt injection and coverage-guided testing feedback to mitigate the effects of faithfulness hallucinations in LLMs and enhance the sufficiency of test cases.

TestART significantly outperforms existing methods, showing an 18% increase in passing rate and a 20% enhancement in coverage rate on tested methods, marking substantial improvements over the capabilities of previous works.

Although TestART is experimented on the ChatGPT-3.5 model, it is superior to the ChatGPT-4.0 model and can be implemented in other LLMs.

TestART shows excellent performance on both open-source datasets and industrial datasets. This indicates that TestART effectively leverages LLMs’ strengths while mitigating their weaknesses, leading to more effective, reliable, and higher-quality unit tests.
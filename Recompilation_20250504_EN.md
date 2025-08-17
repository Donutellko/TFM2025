---
export_on_save:
  prince: true
  html: true

html:
  embed_local_images: false
  embed_svg: true
  offline: false
  toc: undefined

print_background: false
---

# Design of an Evaluation Method for AI Programming Assistants


Donat Shergalis
2025.05.04



## What is missing in many existing benchmarks:

After reviewing existing literature (see article summaries below), we've pointed out the following disadvantages that are common for most, or even all of them.

* Evaluation of the ability to follow additional user instructions:
    - levels of detail in code commenting;
    - instructions on using third-party libraries or avoiding them;
    - instructions on code style;
* Evaluation of the ability to adapt to the code style of the existing codebase:
    - approaches to naming variables and methods - case (camelCase, snake_case, etc.);
    - approaches to code formatting (K&R vs Allman, etc.);
    - use of local terminology (e.g., if the greatest common divisor is referred to as `highest_common_factor` in the provided codebase, the neural network should not use the term `greatest_common_divisor`);
* Evaluation of the ability to make correct decisions without explicitly stating them in the prompt:
    - the neural network should adapt to the code style without requiring it to be mentioned in the prompt;
    - the neural network should not refactor the existing codebase unless explicitly requested;
* Use of a feedback loop based on compilation results, test runs, or LLM evaluation;
* Ability to manually explore the results of each individual task.
    - Rare benchmark allows a user to check individual results. That leads to 
* The cost and time it takees to run a benchmark: 
    - some of the popular benchmarks have hundreds of tests in their datasets;
    - benchmarks often use `pass@k` or `attemt_k` metrics, meaning that each singular test will be run multiple times;
    - recent models implicitly use chain-of-thought under the hood, inflating the number of used tokens per generation;
    - in combined benchmarks many tests are repetitive;
    - there are "saturated" benchmarks (with high average percentage), where many tests don't bring any value, as they are passed by all modern LLMs. 
    - while fine-tuning an LLM, the entire benchmark is run repeatedly.
    
    
After analysing the literature, I formulated the following proposal:

# A Modular Benchmark: More Economic and Eco-frienly

We can also propose a benchmark that will cost less and be more environmentally friendly.

#### Existing problems:
* When developing and fine-tuning a neural network for specific needs, it is often necessary to run the benchmark multiple times. Each run consumes the user's time and money, as well as additional electricity and carbon emissions.
* Many well-known benchmarks primarily compete in the size of their datasets, while only a few publications focus on the quality or diversity of the tasks presented.
* The information a benchmark user receives is often reduced to a single number: the percentage of tests passed from the full set.

## Goals:

#### Make the benchmark modular and customizable

Giving the user the ability to configure the run:
- choose task types (mathematical tasks, refactoring tasks, bug-fixing tasks, etc.);
- configure metrics and testing criteria (objective metrics, external utilities, LLM-judge, etc.);
- choose the number of runs for each task (when using metrics like attempt_k and pass@k);
- choose tasks difficulty (to adjust for benchmark saturation);
- choose programming languages;
- limit the total amount of tasks;
- adjust weights for different testing criteria;
- set a custom instruction for the prompt;
- specify which neural network will be used as the LLM-judge;
- use a feedback loop to improve solutions by providing information about compilation and test runs;
<!-- - introduce a mechanism for managed fluctuation of the test descriptions against test leakage.  -->
- facilitate adding and modification of tests, and adding support for new programming languages; 

#### Obtain more information from each benchmark run

Most benchmarks are limited with a single number as an output — the percentage of tests passed. 

To maximise the information yield for each test, we should use as many ways as possible to and measure and analyse it:
- Objective metrics:
    - percentage of tests passed;
    - execution speed;
    - memory usage by the solution;
- Using external utilities:
    - codestyle check (e.g., Java codestyle, Python pyright);
    - vulnerabilities check (SonarQube);
    - code coverage check (JaCoCo, etc.);
- Subjective metrics (using LLM-judge):
    - code quality;
    - comment quality (not just quantity but also value and relevance);
    - adherence to the existing code style.
- Comparison with the "golden solution" across various parameters.
- Presence of improvements as a result of using the feedback loop.

#### Use different types of tasks
- Code explanation: writing comments or answering specific questions (with short or detailed answers — using LLM-judge);
- Writing high-level documentation;
- Writing code:
    - based on docstrings or based on a detailed descriptions in natural language;
    - from scratch or for an existing codebase;
    - using only standard library or external dependencies;
- Translating code: from pseudocode or another programming language. 
- Refactoring: without instructions, or with additional instructions;
- Optimization: without instructions, or with additional instructions;
- Bug detection and fixing: without instructions, or with additional instructions:
    - logical errors;
    - syntax errors.
- Tasks with a trap, like the interview questions;
- etc.

## Deliverables
- A dataset with tasks (in YAML or JSON format).
- A console application that runs the benchmark with the provided configuration.
- Interactive environments for running the benchmark using Docker (for Java and Python languages).
- A web application that allows:
    - interactive benchmark configuration;
    - adding and editing tasks in the dataset;
    - running the benchmark;
    - exploring the results of the benchmark run for each task.
- A thesis with the following content:
    - what benchmarks for LLMs are and their purposes;
    - existing benchmarks from published articles, with their advantages and disadvantages;
    - other articles comparing benchmarks or highlighting important aspects for creating a useful benchmark;
    - conclusions on what existing benchmarks consider and overlook;
    - problem formulation;
    - description of the proposed solution, its expected advantages and disadvantages;
    - ideas borrowed from existing benchmarks for developing our own, as well as ideas we decided not to adopt;
    - description of the planned product;
    - description of the proposed dataset format;
    - description of task types in it with examples;
    - description of the process of implementing the benchmark and the interactive environment for running it;
    - running the benchmark for some LLMs and compiling rankings based on various criteria;
    - comparison of the benchmark with existing benchmarks in terms of the number and quality of tasks, and performance results.

### Example task in the dataset:
```yaml
tasks:
  - 
    name: highest common factor, implementation from zero
    type: implementation from zero
    difficulty: easy
    area: math
    source: MBPP    # from which benchmark did we get this task
    languages:
        - python
        - java
        - custom    # not every criteria is available for custom languages
    available_parameters: # mostly True/False
        - should-generate-tests # if the LLM should generate tests for the code
        - use-llm-judge         # if we want to check the results with an LLM-judge
        - all-tests-public      # if we want to give the LLM all tests as a reference
        - all-tests-hidden      # if we don't want to give any tests as a reference
        - should-use-libraries  # if we want the solution to use foreign dependencies
    available_criteria:
        - ram_usage         # only for languages that we can run in a sandbox
        - cpu_usage         # only for languages that we can run in a sandbox
        - sonarqube         # for all languages
        - llm-judge-code-quality    # for all languages
        - llm-judge-comment-quality # for all languages
        - java-jacoco       # only for jvm languages
        - java-codestyle    # only for java
        - python-pyright    # only for python
    task:
        common_prompt: |
            Write a function in ${language} to calculate the 
            highest common factor of two numbers. 

            The generated code should only contain one function called `gcd()` 
            and accept two numbers as parameters.
            %if parameters['should-use-libraries']
            You should use a numpy function to achieve the goal.
            %else
            You shouldn't use any foreign libraries to achieve the goal.
            %endif
            
            %if !parameters['all-tests-hidden']
                Here are the tests:
                ${public-tests}
                % if parameters['all-tests-public']
                    ${hidden-tests}
                % endif
            %endif

        languages-specific:
            python:
                description: |
                    ${common_prompt}
                
                public-tests:
                    - 
                    code: | 
                    result = ${solution.function_name}(10, 15)
                    assert result == 5
                hidden-tests:
                    -   
                    code: | 
                      result = ${solution.function_name}(2, 5)
                      assert result == 1
                    - 
                    code: |
                      # tests can validate the source code 
                      # and benchmark parameters, 
                      # for example if we don't want the solution 
                      # to involve additional dependencies
                      if parameters['should-use-libraries'] == True:
                          assert ${solution.code}.contains('numpy')
                      else:
                          assert not ${solution.code}.contains('import')
            java:
                description: |
                    ${common_prompt}
                
                public-tests: |
                    // to do
                hidden-tests: |
                    // to do

    golden-solution:
        java: |
            # to do
        python:
            # to do
        custom: # pseudocode or natural language description
            # to do

    llm-judge-prompt: | 
        You are an experienced interviewer assessing the candidate's solution. 
        Here is the task that was given to the candidate:
        ```
        ${prompt}
        ```
        Based on the given task, the candidate wrote the following solution:
        ```
        ${solution.code}
        ```

        %if parameters['generate_tests']
            And also implemented the following tests:
            ```
            ${solution.tests}
            ```
        %endif

        Based on the provided task and candidate's solution, 
        respond with a YAML that contains numeric evaluations of the 
        following concepts on a scale from 0 to 10:
        ```
        solution_correctness: int
        code_quality: int
        style_quality: int
        %if parameters['generate_tests']
        test_quality: int
        %end_if
        ```

```

# Publications

For each publication, me and my coworkers have extracted key details (short description, conclusion, limitations, used metrics, and benchmark type) and put them in a table, together with additional comments and interecting citations.

![alt text](image.png)

Here, I summed them up, and commented their **Usefulness** for the purposes of our work. 
> 

### Prompting Large Language Models to Tackle the Full Software Development Lifecycle: A Case Study
Link: [https://arxiv.org/abs/2403.08604](https://arxiv.org/abs/2403.08604)  
Published: 13.03.24  

```
@misc{li2024promptinglargelanguagemodels,
      title={Prompting Large Language Models to Tackle the Full Software Development Lifecycle: A Case Study}, 
      author={Bowen Li and Wenhan Wu and Ziwei Tang and Lin Shi and John Yang and Jinyang Li and Shunyu Yao and Chen Qian and Binyuan Hui and Qicheng Zhang and Zhiyin Yu and He Du and Ping Yang and Dahua Lin and Chao Peng and Kai Chen},
      year={2024},
      eprint={2403.08604},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2403.08604}, 
}
```

The authors of the study evaluate the ability of LLMs to conduct the full development process — from design to implementation and writing tests.  

However, their approach **does not test the ability to work with an existing codebase**: the development process is emulated from start to finish in a model similar to Waterfall.  

The tasks and test projects on which the neural networks are tested in this benchmark are quite simple, typical, and very detailed and well-described. Their size is within the context of LLM.  

The results are checked using various methods: automated tests, test coverage checks.  

The tasks themselves, along with the expected solutions, are published on GitHub. As a result, the current task base of the benchmark will quickly become ineffective, as the solutions and tasks will soon be included in the training data for new LLMs. This is called **data leakage**.

**Usefulness:**
> This article describes disadvantages of different benchmarks: they evaluate a small portion of use-cases for AI in software development, such as small self-contained leetcode-style coding task, and are prone to data leakage. And this benchmark goes bigger, by trying to emulate software development cycle end-to-end. 

### SWE-bench: Can Language Models Resolve Real-World GitHub Issues?
Link: [https://arxiv.org/abs/2310.06770](https://arxiv.org/abs/2310.06770)  
Published: 10.10.23  

```bibtex
@misc{jimenez2024swebenchlanguagemodelsresolve,
      title={SWE-bench: Can Language Models Resolve Real-World GitHub Issues?},
      author={Carlos E. Jimenez and John Yang and Alexander Wettig and Shunyu Yao and Kexin Pei and Ofir Press and Karthik Narasimhan},
      year={2024},
      eprint={2310.06770},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2310.06770},
}
```

This well-known study aims to **evaluate the ability of LLMs to solve realistic real-world tasks**, such as GitHub issues.  

The benchmark is presented in two versions:  
1. **Strictly curated tasks** — have clear descriptions and expected results, based on which the validity of the provided solution is assessed.  
2. **Broad list of tasks** — less strictly structured.  

An official website provides a ranking based on the results of a specially secret selection of tasks. This prevents data leakage and avoids training new LLMs specifically to pass these tasks.  

**Advantage**: The benchmark is more reliable compared to DevEval.  
**Limitation**: Evaluates solutions only by strict criteria.

**Usefulness:**
> This article points out that most benchmarks use small academic and leetcode-style coding tasks. Tasks in this dataset are based on existing codebases and are taken from the real world. That can be useful for us while building our own set of tasks. 

### Need Help? Designing Proactive AI Assistants for Programming
Link: [https://arxiv.org/abs/2410.04596](https://arxiv.org/abs/2410.04596)  
Published: 06.10.24  

```
@misc{chen2025needhelpdesigningproactive,
      title={Need Help? Designing Proactive AI Assistants for Programming},
      author={Valerie Chen and Alan Zhu and Sebastian Zhao and Hussein Mozannar and David Sontag and Ameet Talwalkar},
      year={2025},
      eprint={2410.04596},
      archivePrefix={arXiv},
      primaryClass={cs.HC},
      url={https://arxiv.org/abs/2410.04596},
}
```

The authors of the study analyzed the process of working with Copilot and the codebase to identify specific moments when the user might expect **proactive suggestions from LLM**, such as code refactoring suggestions.  

**Usefulness:**
> This article is not very useful for the purposes of creating a benchmark, however, it points out that Copilot's suggestions have to be timed well to be useful. 

### Reading Between the Lines: Modeling User Behavior and Costs in AI-Assisted Programming
Link: [https://arxiv.org/abs/2210.14306](https://arxiv.org/abs/2210.14306)  
Published: 25.10.22  

```bibtex
@misc{mozannar2024readinglinesmodelinguser,
      title={Reading Between the Lines: Modeling User Behavior and Costs in AI-Assisted Programming}, 
      author={Hussein Mozannar and Gagan Bansal and Adam Fourney and Eric Horvitz},
      year={2024},
      eprint={2210.14306},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2210.14306}, 
}
```

Researchers classified various stages of working with Copilot. Their taxonomy includes **12 different states**, such as:  
- Waiting for user input.  
- Waiting for a suggestion from LLM.  
- Reading a suggestion from Copilot.  
- Manually editing code.  

In the study, 21 developers with different levels of experience solved proposed test tasks. The interaction process was recorded on video, and then the video was annotated by the participants themselves.  

**Features of the study**:  
Situations where the developer accepts a suggestion but then deletes or edits it are taken into account. This is because when using Copilot, suggestions are usually given line by line or in small blocks. So the user accepts several suggestions to see the entire block of code, using a linter and syntax highlighting, and then may undo the suggested changes. Most other studies, such as Copilot Arena, do not consider such situations. 

**Results**:  
- About a quarter of the time working with Copilot was spent reading suggestions before accepting or rejecting them.  
- The study is very close to real conditions but is difficult to scale due to manual data annotation.

**Usefulness:**
> This article describes the interaction of a user with his copilot in a way that metrics currently ignore. We can think about how to improve the existing metrics to handle such scenarios for user studies like Copilot Arena. 

### Practices and Challenges of Using GitHub Copilot: An Empirical Study
Link: [https://arxiv.org/abs/2303.08733](https://arxiv.org/abs/2303.08733)  
Published: 09.08.23  

```
@inproceedings{Zhang_2023, series={SEKE2023},
   title={Practices and Challenges of Using GitHub Copilot: An Empirical Study},
   volume={2023},
   ISSN={2325-9000},
   url={http://dx.doi.org/10.18293/SEKE2023-077},
   DOI={10.18293/seke2023-077},
   booktitle={Proceedings of the 35th International Conference on Software Engineering and Knowledge Engineering},
   publisher={KSI Research Inc.},
   author={Zhang, Beiqi and Liang, Peng and Zhou, Xiyu and Ahmad, Aakash and Waseem, Muhammad},
   year={2023},
   month=jul, pages={124–129},
   collection={SEKE2023} }

```

In this empirical study, the authors collected a large list of published problems and questions related to using Copilot on Stack Overflow and GitHub Issues.  

**Main problems**:  
- Difficulties integrating or accessing Copilot (about 50%).  
- Usage limitations (15%).  
- Unsatisfactory code quality (12%).  
- Privacy concerns (8%).  
- Uncomfortable user experience (5%).  

**Usefulness**:  
> Publications related to Copilot usage can inspire the automated evaluation of user experience in a benchmark.

### Do Large Language Model Benchmarks Test Reliability?
Link: [https://arxiv.org/abs/2502.03461](https://arxiv.org/abs/2502.03461)  
Published: 05.02.25  


```bibtex

@misc{vendrow2025largelanguagemodelbenchmarks,
      title={Do Large Language Model Benchmarks Test Reliability?}, 
      author={Joshua Vendrow and Edward Vendrow and Sara Beery and Aleksander Madry},
      year={2025},
      eprint={2502.03461},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/2502.03461}, 
}
```
This study introduces the concept of a **platinum benchmark**.  

The authors carefully examined a number of well-known LLM benchmarks and identified problems:  
- Benchmarks become saturated and stop being used when model results approach the threshold (e.g., 95%).  
- Among the tasks that most often receive incorrect answers, the following were highlighted:  
  - Ambiguously formulated tasks.  
  - Tasks with an incorrect answer as the correct one.  
  - Tasks without a correct answer among the options.  
  - Correct tasks that remain unsolvable for LLMs but are solvable by humans.  

This study does not focus on programming tasks, but the findings can be useful for improving benchmark quality.

**Usefulness**:
> This article points out common problems of the popular benchmarks, such as saturation and incorrectness. We can try to handle saturation issues and implement Quality Assurance for the developed benchmark.

### One-to-many testing for code generation from (just) natural language
Link: [https://www.microsoft.com/en-us/research/wp-content/uploads/2024/09/Improved_MBPP_benchmark-2.pdf](https://www.microsoft.com/en-us/research/wp-content/uploads/2024/09/Improved_MBPP_benchmark-2.pdf)  
Published:  

```bibtex
@inproceedings{uniyal2024one,
  title={One-to-many testing for code generation from (just) natural language},
  author={Uniyal, Mansi and Singh, Mukul and Verbruggen, Gust and Gulwani, Sumit and Le, Vu},
  booktitle={Findings of the Association for Computational Linguistics: EMNLP 2024},
  pages={15397--15402},
  year={2024}
}
```

In this study, the authors improved the existing MBPP benchmark with programming tasks in Python.  

**Main changes**:  
- Tests became less dependent on the expected code structure in the solution, allowing the LLM to choose the method signature.  
- Tasks were reformulated while preserving their essence.  

**Results**:  
The solvability of tasks decreased by 4.5%, indicating that LLMs learned to solve tasks in a specific formulation due to data leakage.

**Usefulness**:
> The article describes a way to tackle data leakage and saturation of the benchmark. 

### HumanEval Pro and MBPP Pro: Evaluating Large Language Models on Self-invoking Code Generation
Link: [https://arxiv.org/abs/2412.21199](https://arxiv.org/abs/2412.21199)  
Published: 30.12.24  

```bibtex
@misc{yu2024humanevalprombpppro,
      title={HumanEval Pro and MBPP Pro: Evaluating Large Language Models on Self-invoking Code Generation},
      author={Zhaojian Yu and Yilun Zhao and Arman Cohan and Xiao-Ping Zhang},
      year={2024},
      eprint={2412.21199},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2412.21199},
}
```

This study approaches the idea of testing the ability of LLMs to work with an existing codebase.  

**Features of the benchmark**:  
- After solving the main task and writing code in Python, the neural network must call the function it wrote to solve the second part of the task.  

**Results**:  
The benchmark identified some LLMs that are less suitable for working with code, as they cannot reuse the code they wrote themselves.

**Usefulness**:
> This benchmark gives an idea of creating multi-step tasks to check the ability of an LLM to work with existing code, written by itself. Also, tasks for the benchmark were created using AI based on existing benchmarks. We might want to borrow that approach and use the prompt they used. 

### User Centric Evaluation of Code Generation Tools
Link: [https://arxiv.org/abs/2402.03130](https://arxiv.org/abs/2402.03130)  
Published: 05.02.24  
```bibtex
@misc{miah2024usercentricevaluationcode,
      title={User Centric Evaluation of Code Generation Tools}, 
      author={Tanha Miah and Hong Zhu},
      year={2024},
      eprint={2402.03130},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2402.03130}, 
}
```

This is a case study evaluating the ability of ChatGPT to write quality code in the R language.  

**Methodology**:  
- The evaluation was conducted using both objective metrics (e.g., attempt_k — the average number of generations to get a correct result) and subjective metrics (accuracy, completeness, conciseness, logic clarity, readability, structure, coverage of parameters, depth of explanation).  

**Features**:  
- Metadata was added to the tasks: task complexity, task type (e.g., encoding a formula, visualizing data), data source (book title, publication year, chapter number).  

**Results**:  
Metadata allows evaluating the neural network's performance not only with a single number but also separately by different groups of complexity and task types.

**Usefulness**:
> Adding metadata for tests can be useful to maximise the information received from the benchmark, as it can allow drawing new conclusions based on the percentage of passed tests in different categories. 
The idea of using multiple different criteria is not common among benchmarks. However, in this benchmark a human evaluates those criteria, and we might want to automate this evaluations using LLM-as-a-judge.  

### SPoC: Search-based Pseudocode to Code
Link: [https://arxiv.org/abs/1906.04908](https://arxiv.org/abs/1906.04908)  
Published: 12.06.19  

```bibtex
@misc{kulal2019spocsearchbasedpseudocodecode,
      title={SPoC: Search-based Pseudocode to Code}, 
      author={Sumith Kulal and Panupong Pasupat and Kartik Chandra and Mina Lee and Oded Padon and Alex Aiken and Percy Liang},
      year={2019},
      eprint={1906.04908},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/1906.04908}, 
}
```


In this article, the authors presented a dataset containing 18,000 programs in pseudocode and C++, written by humans, along with test cases for validation.  

**Features**:  
- Test cases are divided into open and hidden.  
- During training, the neural network used pseudocode, the expected result, and all test cases.  
- During testing, only pseudocode and open tests were provided.  

**Usefulness**:
> We might want to use the approach with open and hidden test cases. We might want to borrow some tasks for rewriting pseudocode.

### Evaluating Large Language Models Trained on Code
Link: [https://arxiv.org/abs/2107.03374](https://arxiv.org/abs/2107.03374)  
Published: 07.07.21  

```bibtex
@misc{chen2021evaluatinglargelanguagemodels,
      title={Evaluating Large Language Models Trained on Code}, 
      author={Mark Chen and Jerry Tworek and Heewoo Jun and Qiming Yuan and Henrique Ponde de Oliveira Pinto and Jared Kaplan and Harri Edwards and Yuri Burda and Nicholas Joseph and Greg Brockman and Alex Ray and Raul Puri and Gretchen Krueger and Michael Petrov and Heidy Khlaaf and Girish Sastry and Pamela Mishkin and Brooke Chan and Scott Gray and Nick Ryder and Mikhail Pavlov and Alethea Power and Lukasz Kaiser and Mohammad Bavarian and Clemens Winter and Philippe Tillet and Felipe Petroski Such and Dave Cummings and Matthias Plappert and Fotios Chantzis and Elizabeth Barnes and Ariel Herbert-Voss and William Hebgen Guss and Alex Nichol and Alex Paino and Nikolas Tezak and Jie Tang and Igor Babuschkin and Suchir Balaji and Shantanu Jain and William Saunders and Christopher Hesse and Andrew N. Carr and Jan Leike and Josh Achiam and Vedant Misra and Evan Morikawa and Alec Radford and Matthew Knight and Miles Brundage and Mira Murati and Katie Mayer and Peter Welinder and Bob McGrew and Dario Amodei and Sam McCandlish and Ilya Sutskever and Wojciech Zaremba},
      year={2021},
      eprint={2107.03374},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/2107.03374}, 
}
```

The authors created the HumanEval benchmark, consisting of short tasks for generating code in Python.  

**Features**:  
- Tasks are described as short docstrings and contain an example solution and tests.  
- The LLM Codex was trained on 300 million and 12 billion parameters based on 150 GB of Python code selected from GitHub repositories.  

**Results**:  
- On the created benchmark, Codex showed results of 13.2% and 37.7% for 300M and 12B parameters, respectively, significantly outperforming other GPTs of that time: for example, GPT-3 showed zero results.  
- The closest competitor was GPT-J-6B with a result of 11.4%, trained on The Pile dataset (800 GB).

**Usefulness**:
> HumanEval is becoming saturated with 96.2% pass@1 rate by gpt-o1-mini, so we might want to borrow tasks from HumanEval Pro. 

### GAIA: a benchmark for General AI Assistants
Link: [https://arxiv.org/abs/2311.12983](https://arxiv.org/abs/2311.12983)  
Published: 21.11.23  

```bibtex
@misc{mialon2023gaiabenchmarkgeneralai,
      title={GAIA: a benchmark for General AI Assistants}, 
      author={Grégoire Mialon and Clémentine Fourrier and Craig Swift and Thomas Wolf and Yann LeCun and Thomas Scialom},
      year={2023},
      eprint={2311.12983},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2311.12983}, 
}
```

This benchmark is not limited to programming tasks and allows evaluating the ability of LLMs to perform complex tasks requiring reasoning, web search, and the use of other tools.  
The benchmark requires performing multiple steps to achieve the goal, thus it also tests the LLM's planning ability.

### Copilot Arena: A Platform for Code LLM Evaluation in the Wild
Link: [https://arxiv.org/abs/2502.09328](https://arxiv.org/abs/2502.09328)  
Published: 13.02.25  

```bibtex
@misc{chi2025copilotarenaplatformcode,
      title={Copilot Arena: A Platform for Code LLM Evaluation in the Wild}, 
      author={Wayne Chi and Valerie Chen and Anastasios Nikolas Angelopoulos and Wei-Lin Chiang and Aditya Mittal and Naman Jain and Tianjun Zhang and Ion Stoica and Chris Donahue and Ameet Talwalkar},
      year={2025},
      eprint={2502.09328},
      archivePrefix={arXiv},
      primaryClass={cs.SE},
      url={https://arxiv.org/abs/2502.09328}, 
}
```

The authors developed a plugin for IDE that offers the user two code completion options from two different LLMs and records which option the user chose.  

**Features**:  
- Evaluates the use of Copilot by real users on real tasks.  
- LLM comparison is done in pairs in a tournament format.  
- The benchmark does not provide an absolute numerical value for each model, only a comparative ranking.

**Limitations**:  
- The need to wait for two completions slows down the work process.  
- The benchmark is used only in one scenario — code completion in small fragments.

**Usefulness**:
> This is the most realistic benchmark, as it uses real user interactions with their code bases. We might want to use their rating as a reference. 

### InterCode: Standardizing and Benchmarking Interactive Coding with Execution Feedback
Link: [https://arxiv.org/abs/2306.14898](https://arxiv.org/abs/2306.14898)  
Published: 26.06.23  

```bibtex
@misc{yang2023intercodestandardizingbenchmarkinginteractive,
      title={InterCode: Standardizing and Benchmarking Interactive Coding with Execution Feedback},
      author={John Yang and Akshara Prabhakar and Karthik Narasimhan and Shunyu Yao},
      year={2023},
      eprint={2306.14898},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2306.14898},
}
```

In this study, a framework was developed for creating benchmarks in an interactive environment in Docker using a feedback loop.  

**Features**:  
- The neural network gets several opportunities to change the code based on the results of the previous attempt.  
- For each programming language, a separate interactive environment needs to be developed.  
- Evaluation of the LLM's ability to act as an autonomous agent or as a copilot in a scenario when the user provides it with compilation and execution errors to fix.  

**Usefulness**:
> We can use this framework with its Docker interactive environments for Python as a base to create our interactive environments for other languages. 

### Benchmarks and Metrics for Evaluations of Code Generation: A Critical Review
Link: [https://arxiv.org/abs/2406.12655](https://arxiv.org/abs/2406.12655)  
Published: 18.06.24  

```bibtex
@misc{paul2024benchmarksmetricsevaluationscode,
      title={Benchmarks and Metrics for Evaluations of Code Generation: A Critical Review}, 
      author={Debalina Ghosh Paul and Hong Zhu and Ian Bayley},
      year={2024},
      eprint={2406.12655},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2406.12655}, 
}
```

This critical review describes well-known metrics and benchmarks.  

**Features**:  
- The authors created a summary table with the results of various LLMs on different benchmarks.  
- The article helps to choose useful metrics and compare one's own benchmark with others.

**Usefulness**:
> This article features some of the most popular benchmarks and metrics for LLMs, and classifies them. We can use this article to classify our benchmark, and to find aspects that are covered by other benchmarks and are not covered by ours. And also to justify the choice of the metrics for our benchmark.

## 
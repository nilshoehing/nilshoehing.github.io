---
layout: post
title: The Ideal Benchmark
date: 2025-11-11
featured_image: /assets/images/benchmark-red.webp
last_modified_at: 2025-11-11
---
# Benchmarks
A benchmark is a systematic test of capabilities, usually automated, that compares the performance of different models. The main capabilities we care about are understanding queries and producing correct responses, but we can also test auxiliary capabilities like helpfulness, robustness, ethical behavior, compliance with rules, and more. Benchmarks are very similar to evaluations, but the latter are typically tests that ensure the quality of a specific product within a company and are therefore composed of test cases on many levels (unit tests, capability tests, API tests, etc.) instead of only capability tests.

Benchmarks are necessary for measuring progress, comparing models with each other and ablating model components. When working with non-deterministic systems like AI, it should never be assumed that things just work. They must always be tested empirically.

Benchmarks typically consist of a [dataset](#dataset) with inputs and the desired ground-truth responses and at least one [metric](#metrics) that defines how to compare the model outputs with the correct responses to calculate a score. Additionally [prompts](#prompts) are used to guide the models in processing the inputs. After running the benchmark [code](#code), further [analysis](#analysis) reveals where and why models fail. [Documentation](#documentation) helps users understand what the benchmark actually tests for.

<p align="center">
<img src="/assets/images/idealbenchmark/overview.svg" alt="Benchmark overview" width="768">
</p>

# Dataset
The main decisions you have to make regarding your data are:
- [Likely vs. Unlikely Data](#likely-vs-unlikely-data)
- [New vs. Recycled Data](#new-vs-recycled-data)
- [Real vs. Synthetic Data](#real-vs-synthetic-data)
- [Dataset Size vs. Quality](#dataset-size-vs-quality)
- [Difficulty](#difficulty)
- [Dataset Splits](#dataset-splits)

### Likely vs. Unlikely Data
#### Importance of Unlikely Data
Benchmarks should contain both likely cases and unlikely cases. As in software testing the important tests are for outliers and edge cases, since they are expected failure points and should therefore be tested rigorously. When you don't include those cases in your benchmarks, you'll get a distorted image of progress. 

Since training data usually follows the probability distribution of data found in the wild, most or all of it covers the likely cases. If we then test the models only with likely cases, we don't know whether the model fully understands the concepts we're testing for or whether it has learned a shortcut. For example, assume you give a chatbot an image of a lamp on a table, you ask where the lamp is, and it answers correctly. Then you still can't conclude that it understands the concept of "on", because you only tested a likely case. Lamps are usually on tables and not under tables. Even a text-only language model (without any image processing) could have given you that response simply because the sentence "a lamp on a table" is much more likely than the sentence "a lamp under a table". 

The more informative test case is therefore the one where the unlikely answer is correct. If models get those unlikely cases wrong even though they get the likely ones correct, we know they don't understand the concept being tested and are instead relying on shortcuts from their training data. In this case, the shortcut would be memorizing that lamps are commonly on tables and not under them. If models get those unlikely cases right, we can be more certain that they actually understand the concept we tested for because the model's decision boundary correctly lies between the two test cases. However, since there may be other shortcuts that we did not think of, we can't be fully certain. 

Unlikely cases are often omitted from benchmarks, because gathering them is hard and requires manual effort. Domain experts can greatly assist in finding them since they will be aware of unique caveats in their area of expertise.

#### Contrastive Benchmarks
Contrastive benchmarks contain tuples of multiple questions paired with multiple answers. The questions only differ in small details and for each question exactly one of the answers is correct. A contrastive design like in [VQA v2](https://arxiv.org/abs/1612.00837), [Winoground](https://arxiv.org/abs/2204.03162) or [RocketScience](https://arxiv.org/abs/2509.02175) encourages including both likely and unlikely cases. If you can ensure that your data will contain rare cases in another way that also works, but thinking in the contrastive framework generally helps. Each contrastive test case in the benchmark should be clean and isolated: test only one specific concept to make it easier to reason about how a model could game the test.

Always consider shortcuts the models might use and take into account the tasks that are easy for them. Based on this, would the images shown below be a good contrastive pair to test spatial understanding? No, because vision-language models have been excellent at detecting the presence of objects for years. They only struggled with spatial reasoning. So the question below is trivially solvable even for an old model like [CLIP](https://arxiv.org/abs/2103.00020) because the second image simply lacks a red ball. To fix it, add the red ball somewhere in the second image, just not inside the bin.
<p align="center"><strong>"Which image shows a red ball in a black bin?"</strong></p>
<div align="center">
<img src="/assets/images/idealbenchmark/bin1.jpg" alt="Beetroot image 1" style="width: 45%; margin-right: 2%;">
<img src="/assets/images/idealbenchmark/bin2.jpg" alt="Beetroot image 2" style="width: 45%; margin-left: 2%;">
</div>
<p align="center">
<em>Would this be a good contrastive pair if we want to test for spatial understanding?</em>
</p>

So while the contrastive format does not require this, aim for minimally contrastive pairs, that differ by the smallest change that should alter the correct answer. Even when your benchmark is not explicitly contrastive, include minimally contrastive samples to test that the models' decision boundaries lie exactly where you intend.
### New vs. Recycled Data
New data that has not yet been published on the internet is ideal, since otherwise models may have been trained on it already. Many benchmarks recycle data from older datasets, which makes high performance on them less indicative of genuine capability. 

Recycling datasets often involves filtering existing data. While semantic similarity search can help find contrastive pairs if you have enough data, the unlikely cases often simply don't exist in the datasets.

### Real vs. Synthetic Data

The data should be as real as possible. What qualifies as real depends on your context - if you are testing how good models are on video games, then video games are "real data". For a chat bot the most real data would be actual user queries along with their intended responses. This similarity to the data that will actually be input by model users is important since small changes to the inputs of neural networks (like changing the resolution of an image or mirroring) can have big effects on the outputs. ([Duran et al.](https://arxiv.org/abs/2502.09460)) As models are supposed to perform well on real data in the end, you should also test on real data. Otherwise your benchmark won't be representative and will lose it's purpose. Domain experts can help both acquire real data and assess which data best represents the domain. When collecting real data, ensure it's broadly representative, for example including daytime and nighttime conditions, diverse environments, low-quality inputs, etc.

Synthetic data can be produced by rule-based systems or by AI generators. Rule-based systems generally produce schematic data, because all desired diversity needs to be explicitly encoded in the logic. Real-world variation is difficult to replicate programatically. Since the goal of testing is to verify that the models perform well on real, varied data in all it's variation, the usefulness of rule-based synthetic data for evaluation is questionable.

AI generators can synthesize data in two ways: by editing existing samples or by producing new ones from scratch. Both approaches share the same limitation - the generator must already understand the semantics of the task. Therefore, they are unsuitable for creating test data for frontier capabilities, since the generators themselves lack deep understanding of the content. However, for tasks within the generator's capabilities you may be able to produce high-quality synthetic data at low cost. Especially when collecting real data is prohibitively expensive, this can be a good compromise.

Another caveat is that generative models tend to produce mostly likely cases. Until very recently, for instance, image generators would only produce images of people on chairs even when asked specifically for "a chair on a person". Moreover, synthetic data quality can be inconsistent. Generated images for example, may show visible artifacts or poor compositional understanding. If quality is your top priority you should still rely on real data. Especially if synthetic data is already part of your training pipeline, avoid testing on similar synthetic samples or you won't be able to tell whether a model is overfitting on them.

All of this only applies only to fully automatic generation. A human-in-the-loop system can yield better results, but at a higher cost in effort and time.

When generating synthetic data, "seed" the model with explicit variations to encourage diversity. Create a detailed list of dimensions you want to cover like "long vs. short text", "spelling mistakes vs. perfect spelling" or "bright vs. dim lighting" and iterate through them while prompting your data generator. ([Husain](https://hamel.dev/blog/posts/evals-faq/what-is-the-best-approach-for-generating-synthetic-data.html))




### Dataset Size vs. Quality
#### Size
Dataset size is important because benchmarks need to be reproducible. This means that rerunning a model on a benchmark should yield consistent scores within an acceptable  statistical margin. Since the models are usually non-deterministic, benchmarks must be large enough to guarantee low variance across runs.

Some researchers argue that we need large, super-low variance benchmarks to measure the exact ranking of models on leaderboards. If instead we treat benchmarks primarily as tools for assessing current capabilities and identifying areas for improvement, then a variance of one or two percentage points is perfectly tolerable. Differences of that size are not meaningful unless you're at the very top of the benchmark, where small gains reduce the error rate significantly. However at that point you should already have built a new and harder version of your benchmark. Achieving a variance of one or two percentage points often requires only a few hundred samples. If you require more fine grained analysis then of course a larger dataset will naturally reduce the variance further.

The benefits of a small benchmark are that it can be run quickly and cost-effectively, enabling frequent evaluations of deployed models to ensure ongoing quality. Smaller benchmarks are also easier to replace when they approach saturation or when they have been used so often that you suspect overfitting. I recommend choosing a larger dataset only when it allows you to cover greater diversity and a broader range of scenarios. If expanding the dataset would simply add redundant or highly similar data it's better to keep it smaller and more focussed.


#### Quality
You can ensure much higher quality by keeping the benchmark small. By being actively involved in data collection and labelling rather than delegating it entirely, you can maintain high standards. In contrast, crowdsourcing large datasets has historically resulted in extremely high mislabelling rates. Large-scale synthetic data without human verification is also prone to be low in diversity and contain obvious errors. This kind of low quality data introduces an invisible ceiling below 100% which models cannot surpass through genuine improvement and can only exceed by overfitting to the benchmark. 




### Difficulty
If you want to publish at a conference, then your benchmark needs to be challenging for current state-of-the-art models so that there is significant room for improvement. If you want to ensure quality within a company, you need to diagnose performance changes in both directions (improvement and regression), which isn't possible with a benchmark where every model scores at the very bottom. In that case, models should perform somewhere around the midpoint between random chance and the highest possible score. Even a saturated benchmark, that might be criticised for being too easy, can still reveal meaningful insights, such as performance gaps between model classes.

You can increase the difficulty of your benchmark by creating novel tasks that current models haven't been trained on or by adjusting the ratio of likely to unlikely data. Introducing new question and answer formats that models are not used to also helps, for example, adding a "none of those" option to multiple-choice questions can reduce model accuracy. ([Elhady et al.](https://arxiv.org/abs/2502.18316))

Even if you want to build a hard benchmark, you should aim to minimize ambiguity in the test cases. Ambiguity is a common factor that increases difficulty unintentionally and should be avoided since it can render parts of your benchmark unsolvable (except for models that memorize it). The best way to ensure low ambiguity is by measuring human performance with domain experts. If they perform well the benchmark is solvable which implies that it has low ambiguity. This approach doesn't apply to benchmarks testing for superhuman performance, but since models still lag behind experts in many areas, it remains useful. Benchmarks that don't report human performance should raise suspicion and prompt careful examination of the data for solvability. Less empirical methods of checking ambiguity include reviewing data for contradictions or unclear instructions together with colleagues.

One reason researchers often avoid running human baselines is that crowd workers are notoriously inattentive and significantly underperform compared to motivated experts. Today they might even use AI to respond. Therefore, I recommend selecting trusted individuals who were not involved in the creation of the benchmark to perform establish the human baseline.


### Dataset Splits
With some exceptions, such as [ARC-AGI](https://arcprize.org/arc-agi), modern benchmarks no longer include training sets. This is because we moved from specialist fine-tunes to generalist models which need to perform out of the box. Also we want to assess the actual performance of those generalist models not their potential performance after fine-tuning on that benchmark. Another common perspective is that fine-tuning on a benchmark's train set can cause models to pick up dataset-specific shortcut solutions, resulting in high scores that are not indicative of performance outside the benchmark. Requiring generalization from a different training set to a new test set helps mitigate the risk of results being limited to a single benchmark.

Keeping additional slices of the dataset to yourself as private holdout test sets can be a good strategy (even for internal company benchmarks) to detect overfitting to the public test set.


# Prompts
In benchmarks a prompt is like a template that specifies how to process the input data and how to format the response. Choose a format that allows you to easily extract the output. One-shot and few-shot prompts include examples of input and desired output to help models understand the task. You then append the input data to the prompt and feed it into the models.

To develop an effective prompt, create a first draft either manually or with the help of an LLM. Test it across multiple models to see whether they understand it. Iterate until most models respond without asking clarifying questions or showing confusion. This prompt crafting stage is necessary, although it arguably can introduce bias toward some models. In coming years, as models become better at following prompts precisely and returning answers in the requested format, we can hopefully skip this stage.

<p align="center">
<img src="/assets/images/idealbenchmark/prompts.svg" alt="Prompt types" width="768">
</p>



# Metrics
If you want companies and papers to adopt your benchmark, it should typically report a single primary metric. Otherwise you are making it too difficult to include you in their tables. Additional metrics can of course be used for internal analysis. Metrics will be covered in detail in a separate blog post. 

# Code
For multiple choice questions make sure the correct responses are evenly distributed across all answer positions. This is necessary because models exhibit biases toward certain positions, often because those positions are more likely to be correct. Contrastive benchmarks alternate the position of the correct answer naturally.

In the past we would create a controlled, fair environment with equal hyperparameters to compare models, since most were open source. Today many top-performing models are closed source or too large to run locally and come with a variety of hyperparameters. Therefore it has become standard to use each model's default hyperparameters as much as possible. Exceptions include controlling reasoning length in chain-of-thought models to give all of them similar space to think. Be aware that open source models can perform very differently depending on the provider (due to incorrect settings or implementation) and intermediary platforms like OpenRouter may also introduce bugs affecting performance. If your budget allows, running multiple iterations per model and reporting the mean and standard deviation is highly recommended. It is also advisable to log all details (output, time, number of tokens, errors, reasoning traces, model hoster) for each API query, to enable broad analysis of the results without having to rerun models. 

To make your benchmark easy to use, implement it in a popular evaluation framework like [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness), [helm](https://github.com/stanford-crfm/helm) or [openbench](https://github.com/groq/openbench).




# Analysis
To facilitate further analysis it is helpful to label your dataset with fine-grained tags specifying the type of each item. After running the benchmark you may then stratify the results by label to identify the hardest tasks. Examining those samples can provide insights where and potentially also how to improve the next generation of models. With newer reasoning models that reveal their chain-of-thought, analyzing failures becomes more feasible and provides additional guidance for improvements.

You can also filter by individual words in the inputs to discover interesting patterns. For example, the [ARO benchmark](https://arxiv.org/abs/2210.01936) which tests spatial understanding, found that models tended to perform well when questions involved spatial relations like "on" and "under", but for "left" and "right", models performed at chance. "Left" and "right" are the only unbiased spatial relations where any object has equal probability of being on either side of another object. In contrast, relations like "on" are biased since some objects are way more likely to be on top of others than underneath. This suggests that models were relying on those shortcut associations for certain spatial relations rather than genuine spatial reasoning. Note that this benchmark is a few years old and newer reasoning models do understand spatial relations.


# Documentation
Add a clear statement of your benchmark's scope. Consider the implicit decisions made during data collection: which countries or contexts were included? Did you test the broad phenomenon claimed in the title or only a narrower subset? These distinctions matter because models do not necessarily generalize across contexts such as Western vs. non-Western objects. Failing to test for this often reveals that models did not generalize as broadly as assumed later.
Proper documentation helps users interpret the significance of benchmark results for their use case. It is especially important for non-technical stakeholders, so they can understand both what was tested and what was not.





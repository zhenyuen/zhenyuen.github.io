---
layout: distill
title: my reflections as a software engineer (intern)
description: a blog post detailing my thoughts on what it means to be a good software engineer, and how I am learning to become one.
giscus_comments: false
date: 2023-09-06

authors:
  - name: Zhen Yuen Chong
    affiliations:
      name: University of Cambridge

# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: About
  - name: My background
  - name: The assessment
  - name: The video
  - name: Wrap-up

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }

---
## About

As my internship at Visa draws to a close, I have some reflections on myself as a software engineer (SWE). This post isn't about providing advice but rather sharing my thoughts on what it means to be a good SWE (in my opinion), and how I'm working towards that goal. If there are any experienced readers out there, please feel free to get in touch. I would love to hear about your thoughts :)


## My background

During my time at ARM, I had the privilege of working with the build team, under the guidance of Matt, a principal software engineer. It's hard to put into words how much I've learned from him and the team in just a few months -- and it was what inspired me to become not just a good software engineer, but a great one. Working with the build team exposed me to the realities of software engineering. In school, we're taught how to write the 'perfect' code along with proper documentation. However, in the industry, when ever-changing business requirements and tight deadlines come into play, good coding practices can start to vanish, leaving what's known as "spaghetti code". If it works, it works. The build teams' responsibility includes fixing or refactoring codebases before they are built or compiled to ensure proper sharding and caching for remote build execution. There, I've gotten a slight taste of what constitutes good or bad code.

I believe a good software engineer can balance the trade-off between writing quick and dirty code versus striving for perfection. This requires the ability to recognize patterns in code structure and apply the appropriate design patterns, keeping things decoupled and modular while minimizing abstractions. Here, I won't focus on this aspect but rather on another -— writing clean and elegant code with the help of data structures and algorithms (DSA) knowledge (here, I'll be talking about graph problems specifically).

Having a strong understanding of data structures and algorithms forms the foundation of a good software engineer. Unfortunately, many people only study DSA to pass SWE interviews and never apply it in their work. My internship at Visa as a backend engineer, a role that comprises a large majority of SWE positions, shed light on why this happens. We often rely on well-written libraries, and with looming deadlines, it's hard to justify writing proprietary modules when suitable and sufficiently good alternatives are available. It was only when I attempted a timed online assessment in C++ and watched [Jane Street's mock SWE interview with Grace and Nolan](https://www.youtube.com/watch?v=V8DGdPkBBxg&t=1644s&pp=ygULamFuZSBzdHJlZXQ%3D) on YouTube that I realized I had fallen into this trap too.

Before delving further into the assessment, if you'd like to see how DSA helps in writing clean and elegant code, try [this Leetcode problem](https://leetcode.com/problems/valid-number/). My initial solution, like many, contained numerous if-else statements to handle edge cases. However, a much more elegant solution treats the problem as a finite state machine, resulting in a significantly cleaner solution. While developers frequently use Regex expressions in our work, most people fail to appreciate the beauty, elegance, and efficiency behind their implementation, which also leverages the same underlying idea (feel free to look up finite automata :)).


## The assessment

The assessment involved taking a list of three integers, corresponding to `job_id`, `job_duration`, and `next_job_id`, and printing the list of job chains with their details to stdout (for those who are curious, the overall goal is to format the outputs from a concurrenct logging tool). Each job chain includes details like `starting_id`, `ending_id`, `total_duration`, and `average_duration`. A job chain is essentially a sequence of jobs that terminates when `next_job_id=0`. I was provided with one simple example: `[[1, 2, 3], [2, 5, 0], [3, 4, 0]]`. In this case, we have two job chains: `[[1, 2, 3], [3, 4, 0]]` and `[[2, 5, 0]]`. In case of bad inputs, the program should print `Malformed inputs`.

My initial approach was to parse the input and apply DFS, starting from job id `1` (for this example), traversing related jobs and summing up the durations (alternatively, you can view it as a linked list). Immediately, I begun coding right away. However, as I progressed, I began to consider malformed inputs. Bad characters? I shoud be able to handle that in the parsing stage with -- you guessed right -- if else statements. 

Then it hit me. What happens if there is a loop in the specified job ids? What happens if the starting job comes AFTER the intermediate or final job in the input list (i.e., `[2, 5, 0], [3, 4, 0]], [[1, 2, 3]`). There were no assumptions given, no additional examples, and the possibilities were endless. As the concerns piled on, I had to modify my code to account for these scenarios as it slowly devolved into a huge mess. I could no longer print out the job chains as required as I traversed the data -- if an error occured halfway, I should not have printed out the valid job chains (if any) in the first place. Hence, I had to move everything into a `std::stringstream` buffer instead (and flush it in the end if no error occurs), which also increased the overally memory complexity and slowed things down as string processing are slow in general. Unsurprisingly, I failed the assessment.

Right after it ended, I immediately realized I could have used a directed acyclic graph and traversed the nodes in topological order. When I received the question under time pressure, my first instinct as an inexperienced SWE was to use a quick and dirty solution, then slowly refining the code as what is usually done in an agile setup. I started typing my new solution which was much cleaner. It was then I developed my own approach to building software. In my (limited) experience, most applications can be brokened down into two parts:

1. The processing part that handles inputs, processes them, and produces outputs. This is where our knowledge of DSA (Data Structures and Algorithms) can be often applied.

2. The framework which serves as the glue that ties everything together, including all the processing. While DSA can be applied at times, usually it's the use of appropriate design patterns and abstractions that benefit the most.

Focusing on (1), I adopted a 4-stage process for most of my recent works:

1. Parse the inputs, and identify what constitutes a graph node and their relationships. In most processing workloads, more often than not we will find that the given problem is reducible into a graph problem, you just have to look hard enough :)
2. Define the graph or components.
3. For the desired operation, build the required computational graph/sequence -- traverse the nodes and perform the necessary checks (e.g., cycle detection). Rather than constructing the desired output as we traverse the graph, leave that to later, and just output the correct traversal sequence (if valid). This is an important point, as I will explain later.
4. Execute the computation using the pre-computed traversal sequence.

These 4 steps are applicable to most problems, albeit doing some necessary adjustments depending on the problem. Applying this metholodgy to my assessment:

1. Each node and edges can be uniquely identified by the `job_id` and the corresponding `next_job_id`.
2. When defining the graph, we leave out `job_durations` for now, as they do not matter when building our computational graph (note that the duration are NOT edge weights).
3. Traverse the graph in a topological order (DFS) and perform cycle detection at the same time. In addition, cache the traversal sequence.
4. Once we have the correct and valid sequence, just use a basic for-loop and perform the required operations -- summing up the durations, computing averages, etc.

A sample code is provided in my [Git repository](https://github.com/zhenyuen/assessments/tree/main)

You can certainly combine steps 3 and 4 together -- traverse and process at the same time -- that is what most Leetcode problems do, as they are just toy examples. In practice, however, significant processing and aggregations are often necessary, resulting in code that may appear messy, especially when recursion is involved. Storing all intermediate processing on the stack is not ideal, and ideally, no processing should occur if the build is bound to fail initially. This is why I prefer separating both steps. In addition, try keeping the graphs in their most minimal representations -- we only need sufficient information to build and verify the computation graph, nothing more.


## The video

So, how does [the video](https://www.youtube.com/watch?v=V8DGdPkBBxg&t=1644s&pp=ygULamFuZSBzdHJlZXQ%3D) relate to my example?

Here, Nolan was tasked with implementing a unit conversion tool during his interview. For instance, how do you convert 1 cm into inches? It seems straightforward, doesn't it? When I first watched the video a few months ago, my initial thought was to use maps and if-else statements. Why not? 1 inch equals 2.54 cm, so we could create a map specifying the multiplication/division factor. When Nolan first suggested approaching it as a graph problem and using BFS on his first attempt, I questioned if it was really necessary.

Looking back at the video now, I have a different perspective. When you delve deeper into the problem, you'll appreciate the elegance of his approach. As we expand the number of available units for conversion or even define custom units or metrics, his approach allows us to extend his code naturally (adding more nodes/edges). This is preferable to maintaining a map with an exponentially growing number of entries and a multitude of if-else statements.

Some of my points about separating the construction of the computation graph from its execution can be observed in the video. At one point, Nolan encountered minor challenges when incorporating processing into BFS, such as determining what value to return. Additionally, he was using Python, whereas other languages like C++ may not appear as elegant when returning tuples with many elements of varying types.

Under the same [repository](https://github.com/zhenyuen/assessments/tree/main/si_imperial_converter), I've added my own take on the problem -- although I have a much smaller set of conversion units but the same underying logic. You may notice a CMakeLists.txt file that is incomplete, with numerous debugging statements controlled by a global debugging flag. I'm still in the process of learning how to write proper test benches in C++ and utilize CMake effectively without relying on my preferred Makefiles. Therefore, I apologize for any inconvenience this may cause.


## Wrap-up

A code is truly bug-free only when it can be formally proven with mathematics.

Although this doesn't often apply in practical scenarios, it doesn't mean we shouldn't strive for it. By breaking down our code into smaller components and employing well-proven algorithms to tackle specific tasks and processes, we inch-closer to producing higher-quality, error-free code.

During my time at Visa, one of my favorite projects involved creating a tool that generated configuration files dynamically at runtime. Through the use of Regex expressions, lookup tables, and a straightforward depth-first search (DFS) approach, I successfully transformed what initially appeared to be an $O(n^3)$ problem into an $O(n^2)$ solution with some understanding of DSA (Data Structures and Algorithms).

I hope this write-up wasn't too lengthy, and I hope it sheds light on my thought process and journey towards becoming a better software engineer. Thank you for reading through to the end!
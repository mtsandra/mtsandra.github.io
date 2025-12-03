---
layout: post
title:  "ChatGPT Prompt Engineering"
date:   2023-05-01
tags: tech tutorial
published: false
categories: beginners-guide
description: 'Quick notes based on the course taught by Andrew Ng and Isa Fulford'
---
Over the weekend I watched the quick [course](https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/) put out by Andrew Ng and Isa Fulford about ChatGPT Prompt Engineering for developers. In the video series, they showed a lot of examples that inspired me to come up with my own application ideas, so I definitely recommend watching the course. It took me around 2 hours to finish the whole series. Below are notes for quick reference and reminders. 

### Types of LLMs

**Base LLM vs Instruction Tuned LLM**

- Base LLM: predicts next word, based on text training data
    
    - Example:
        
        - Input: What is the capital of France?
            
        - Output: What is France’s largest city? What is France’s population?
            
- Instruction Tuned LLM: tires to follow instructions
    
    - Example:
        
        - Input: What is the capital of France?
            
        - Output: The capital of France is Paris.
            
    - Start off with a base LLM that’s been trained on a huge amount of text data and further fine tune it with inputs and outputs that are instructions and good attempts to follow these instructions
        
    - Then further refined by RLHF -> Reinforcement learning with human feedback
        
    - Recommended for most applications to be deveoped
        

### Guideline for Prompting

**First principle**: give clear and specific instructions

- Clear $$ \neq  $$ Short
    
- Tactic 1: Use delimiters to clearly indicate distinct parts of the input
    
    - Delimiters examples: ```, “““, <>, \<tag> \</tag>, : , ---
        
    - Helps avoid prompt injections as well
        
- Tactic 2: Ask for a structured output (i.e. JSON, HTML)
    
- Tactic 3: Ask the model to check whether conditions are satisfied
    
- Tactic 4: Few-shot prompting -> Give successful examples of completing tasks then ask model to perform the task
    

**Second principle**: give the model time to think

- Tactic 1: Specify the steps required to complete a task and ask for output in a specified format
    
- Tactic 2: Instruct the model to work out its own solution before rushing to a conclusion
    

Model Limitations:

- Hallucination: makes statements that sound plausible but are not true
    
    - Reducing hallucinations: first find relevant information, then answer the question based on the relevant information
        

### Iterative Prompt Development

Prompt guidelines

- Be clear and specific the first time
    
- Analyze why result does not give desired output
    
- Clarify instructions, give more time to think
    
- Refine the idea and the prompt with a batch of examples -> build on the initial prompt
    
- Repeat

<img src="/assets/img/blog2023/prompt-engineering/iterative.png" alt='iterative process' width = "100%" style="padding-bottom:0.5em;" />
    

### Summarizing

- Could ask the model to summarize with specific requirements such as word limits and focus area.
    
- Could also extract instead of summarize (summarize may include more information than asked for)
    
- Could put a list of similar texts in a list and use a loop to iterate through the list to do the same type of summarizing
    

### Inferring

- Given a paragraph, infer emotions, or answer questions with specific requirements
    
- Infer topics and could also determine if the text talks about a certain topic
    

### Transforming

- Can be used to do common transformation tasks such as translation, language detection
    
- Could build a universal translator by iterating through messages in different languages in a loop
    
- Could do tone transformation, format conversion (HTML -> JSON, etc.)
    
- Could use spellcheck/grammar check
    
    - Cool library to use to display the edited version with red lines indicating the changes from the original version:
        
        ```python
        from redlines import Redlines
        diff = Redlines(text, response)
        display(Markdown(diff.output_markdown))
        ```
        

### Expanding

- Could expand on existing content based on prompt
    
- Temperature: degree of exploration or randomness of the model
    
    - Temperature = 0: the model will always choose the most likely next word
        
    - Temperature higher: will choose more of the less likely next word
        

### Chatbot

- Build your own chatbot: instead of only giving ChatGPT one message at a time, give it multiple messages and specify roles in JSON format:
    
    ```JSON
    messages = [
        {'role': 'system', 'content': 'You are a friendly chatbot.'}, 
        {'role': 'user', 'content': 'tell me a joke'}, 
        {'role': 'assistant', 'content': 'Why did the chicken cross the road'}, 
        {'role':'user', 'content':  'i don\'t know'} 
        ]
    ```
    
- Add context -> write a helper function that automatically collects context and add input and output responses from the user and chatbot interaction (i.e. adding to the messages list)
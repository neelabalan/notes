## Generative agents

> [Source](https://api.wandb.ai/links/vincenttu/v3tgur4t) - _Vincent Tu_

- **Challenge #1:** Simulating human behavior with agents needs the agent to reason about their experiences/memories. The authors use a memory stream, but using the entire memory stream is inefficient and distracting. How should we go about this?

- **Solution #1:** The memory stream is composed of what the agent sees around him/herself. It can also be behaviors performed by the agent, and interactions with other agents synthesized into memories. The retrieval mechanism factors recency, relevancy/salience, and importance. 

    - Recency: assigning higher priority to memories recently accessed via exponential decay with decay rate 0.99
    - Importance: an absolute importance score is assigned to every memory; an important memory is like getting a job, and an unimportant memory is like eating breakfast
    - Relevancy/Salience: cosine similarity between the text embedding vectors of the memory stream and the query prompt; essentially, which memories are relevant to this query prompt?


- **Challenge #2:** Agents struggle with performing inference with raw memories as context. The issue of having too many memories for inference also arises.
- **Solution #2:** A second type of memory in the memory stream: a reflection. They exist alongside other memories in the memory stream, but they are more abstract and higher-level. Reflections are generated only when a threshold (sum of recent memories' importance scores) is exceeded. 

    The process for reflecting looks like this:
    - Identifying what to reflect on: query the LLM for 100 most recent memories → generate 3 high-level salient questions about the memories
    - Getting context: 3-pronged retrieval from the memory stream for a set of memories to accompany each question
    - Simulating reflection: for each question, generate 5 novel insights
Updating the memory stream: append these to the memory stream


- **Challenge #3:** Agents need to plan over long horizons. LLMs can't do this by just simply passing in lots and lots of context.
- **Solution #3:** Plans are stored in the memory stream and they keep the agent's behavior consistent over time. They are included in the retrieval process. These plans outline a single day and are prompted by the agent's identity/summary description and a summary of their previous day. As the agent goes about its day, this plan will be recursively edited to include more and more detail.  

The first phase of RAG development and experimentation is the preparation phase. During this phase, you first define the business domain for your solution. Once you have the domain defined, you begin the parallel process of gathering documents and sample questions that are pertinent to the domain. The steps are done in parallel because they're interrelated. The questions must be answerable by content in the documents and the documents must answer relevant questions.

> This article is part of a series. Read the [introduction](./rag-solution-design-and-evaluation-guide.yml).

## Determine solution domain

The first step in this process is to clearly define the business requirements for the solution or the use case. These requirements help determine what kind of questions the solution intends to address and what source data or documents help address those questions. In later stages, the solution domain helps inform your embedding model strategy.

## Gather representative test documents

In this step, you're gathering documents that are the best representation of the documents that you use in your production solution. The documents must address the defined use case and be able to answer the questions gathered in the question gathering parallel phase.

### Considerations

Consider these areas when evaluating potential representative test documents:

1. **Pertinence** - The documents must meet the business requirements of the conversational application. For example, if you're building a chat bot tasked with helping customers perform banking operations, the documents should match that requirement, such as documents showing how to open or close a bank account. The documents must be able to address the test questions that are being gathered in the parallel step. If the documents don't have the information relevant to the questions, all the LLM can do is hallucinate.
2. **Representative** - The documents should be representative of the different types of documents that your solution will use. For example, a car insurance document is different to a health insurance or life insurance document. Suppose the use case requires the solution to support all three types, and you only had two car insurance documents, your solution would perform poorly for both health and life insurance. You should have at least 2 for each variation.
3. **Physical document quality** - The documents need to be in a usable shape. Scanned images, for example, might not allow you to extract usable information.
4. **Document content quality** - The documents must have high content quality. There should not be misspellings or grammatical errors. LLMs don't perform well if you provide them with poor quality content.

The success factor in this step is being *qualitatively confident* that you have a good representation of test documents for your particular domain.

### Test document guidance

* Prefer real documents over synthetic. Real documents must to go through a cleaning process to remove personally identifiable information (PII).
* Consider augmenting your documents with synthetic data to ensure you're handling all kinds of scenarios.
* If you must use synthetic data, do your best to make it as close to real data as possible.
* Make sure that the documents can address the questions that are being gathered.
* You should have at least two documents for each document variant.
* You can use LLMs to help evaluate the document quality.

## Gather test queries

In this step, you're gathering test queries that you'll use to evaluate your chunks, search solution, and your prompt engineering. You'll do this in lockstep with gathering the representative documents, as you're not only gathering the queries, you're also gathering how the representative documents address the queries. Having both the sample queries, combined with the parts of the sample documents that address those queries, allow you to evaluate every stage of the RAG solution as you're experimenting with different strategies and approaches.

### Gather test query output

The output of this phase includes content from both the [Gather representative test queries](#gather-test-queries) step, and the [Gather representative test documents](#gather-representative-test-documents) step. The output is a collection containing the following data:

* **Query** - The query.
* **Context** - A collection of all the actual text in the documents that address the query. For each bit of context, you should include the page and the actual text.
* **Answer** - A valid response to the query. The response be content directly from the documents or it might be rephrased from one or more pieces of context.

### Creating synthetic queries

It's often challenging for the subject matter experts (SMEs) for a particular domain to put together a comprehensive list of questions for the use case. One solution to this challenge is to generate synthetic questions from the representative test documents that were gathered. The following is a real-world approach for generating synthetic questions from representative documents:

1. **Chunk the documents** - Break down the documents into chunks. This chunking step isn't using the chunking strategy for your overall solution. It's a one-off step that will be used for generating synthetic queries. The chunking can be done manually if the number of documents is reasonable.
2. **Generate queries per chunk** - For each chunk, generate queries either manually or using an LLM. When using an LLM, we generally start by generating two queries per chunk. The LLM can also be used to create the answer. The following example shows a prompt that generates questions and answers for a chunk.

    ```text
    Please read the following CONTEXT and generate two question and answer json objects in an array based on the CONTEXT provided. The questions should require deep reading comprehension, logical inference, deduction, and connecting ideas across the text. Avoid simplistic retrieval or pattern matching questions. Instead, focus on questions that test the ability to reason about the text in complex ways, draw subtle conclusions, and combine multiple pieces of information to arrive at an answer. Ensure that the questions are relevant, specific, and cover the key points of the CONTEXT.  Provide concise answers to each question, directly quoting the text from provided context. Provide the array output in strict JSON format as shown in output format. Ensure that the generated JSON is 100 percent structurally correct, with proper nesting, comma placement, and quotation marks. There should not be any comma after last element in the array.

    Output format:
    [
      {
        "question": "Question 1",
        "answer": "Answer 1"
      },
      {
        "question": "Question 2",
        "answer": "Answer 2"
      }
    ]
    
    CONTEXT:
    ```

3. **Verify output** - Verify that the questions are pertinent to the use case and that the answers address the question.  

### Unaddressed queries

It's important to gather queries that the documents don't address, along with queries that are addressed. When you test your solution, particularly when you test the LLM, you need to determine how the solution should respond to queries it doesn't have sufficient context to answer. Approaches to responding to queries you can't address include:

* Responding that you don't know
* Responding that you don't know and providing a link where the user might find more information

### Gather test queries guidance

* Determine if there is a system that contains real customer questions that you can use. For example, if you're building a chat bot to answer customer questions, you might be able to use customer questions from your help desk, FAQs, or ticketing system.
* The customer or SME for the use case should act as a quality gate to determine whether or not the gathered documents, the associated test queries, and the answers to the queries from the documents are comprehensive, representative, and are correct.
* Reviewing the body of questions and answers should be done periodically to ensure that they continue to accurately reflect the source documents.

## Contributors

* [Ritesh Modi](https://www.linkedin.com/in/ritesh-modi/)
* [Rob Bagby](https://www.linkedin.com/in/robbagby/)
* [Ryan Pfalz](https://www.linkedin.com/in/ryanpfalz/)
* [Raouf Aliouat](https://www.linkedin.com/in/raouf-aliouat/)
* [Randy Thurman](https://www.linkedin.com/in/randy-thurman-2917549/)
* [Prabal Deb](https://www.linkedin.com/in/prabaldeb/)

## Next steps

> [!div class="nextstepaction"]
> [Chunking phase](./rag-chunking-phase.yml)

## Related resources

* [Automate document processing by using AI Document Intelligence](/azure/architecture/ai-ml/architecture/automate-document-processing-azure-form-recognizer)
* [Get started with the Python enterprise chat sample using RAG](/azure/developer/python/get-started-app-chat-template?tabs=github-codespaces)

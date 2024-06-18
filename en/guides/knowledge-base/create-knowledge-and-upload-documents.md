# Creating a Knowledge Base & Uploading Documents

### 1 Creating a Knowledge Base

Click on Knowledge in the main navigation bar of Dify. On this page, you can see your existing knowledge bases. Click **Create Knowledge** to enter the setup wizard:

<figure><img src="../../.gitbook/assets/create-knowledge.png" alt=""><figcaption><p>Creating Knowledge</p></figcaption></figure>

* If you have already prepared the files, you can start by uploading them;
* If you have not prepared any documents yet, you can first create an empty dataset;

<figure><img src="../../.gitbook/assets/create-knowledge-2.png" alt=""><figcaption><p>Creating Knowledge Base</p></figcaption></figure>

{% hint style="info" %}
If you choose to use an external data source when creating a dataset, the type of knowledge cannot be changed. This is to prevent difficulties in managing datasets caused by having multiple data sources in a single knowledge base. If you need to use multiple data sources, it is recommended to create multiple knowledge.
{% endhint %}

***

### 2 Uploading Documents

**Steps to upload documents into Knowledge:**

1. Select the document you need to upload from your local files;
2. Segment and clean the document, and preview the effect;
3. Choose and configure Index Mode and Retreival Settings;
4. Wait for the chunks to be embedded;
5. Upload completed, now you can use it in your applications 🎉

**Limitations for uploading documents:**

* The upload size limit for a single document is 15MB;
* The maximum number of files for a single batch upload is 20;
* Different [subscription plans](https://dify.ai/pricing) for the SaaS version limit **batch upload numbers, total document uploads, and vector storage**;

### 3 Segmenting and Cleaning

**Segmenting**: Large language models have a limited context window, usually requiring the entire text to be segmented and then recalling the most relevant segments to the user’s question, known as the segment TopK recall mode. Additionally, appropriate segment sizes help match the most relevant text content and reduce information noise when semantically matching user questions with text segments.

**Cleaning**: To ensure the quality of text recall, it is usually necessary to clean the data before passing it into the model. For example, unwanted characters or blank lines in the output may affect the quality of the response. To help users solve this problem, Dify provides various cleaning methods to help clean the output before sending it to downstream applications.

Segmentation and cleaning support two configuration strategies:

* Automatic mode (to be phased out)
* Custom mode

<figure><img src="../../.gitbook/assets/custom-chunk-settings.png" alt=""><figcaption></figcaption></figure>

In custom mode, users can configure chunk settings and cleaning settings according to different document formats and scenario requirements.

**Segmentation rules:**

* Segmentation identifier, set an identifier such as "\n", and the system will segment the text when the identifier appears in the text;
* Maximum segment length, segment based on the maximum character limit of the text, forcibly segmenting when exceeding this length;
* Segment overlap length, set the number of overlapping characters between segments, it is recommended to set it to 10-25% of the segment length, which helps retain semantic relevance between segments and improves recall results during multi-segment recall.

**Preprocessing rules:**

* Replace continuous spaces, newlines, and tabs;
* Delete all URLs and email addresses;

***

### 4 Optional ETL Configuration

In production-level applications of RAG, to achieve better data recall, multi-source data needs to be preprocessed and cleaned, i.e., ETL (extract, transform, load). To enhance the preprocessing capabilities of unstructured/semi-structured data, Dify supports optional ETL solutions: **Dify ETL** and [**Unstructured ETL**](https://unstructured.io/).

> Unstructured can efficiently extract and transform your data into clean data for subsequent steps.

ETL solution choices in different versions of Dify:

* The SaaS version defaults to using Unstructured ETL and cannot be changed;
* The community version defaults to using Dify ETL but can enable Unstructured ETL through [environment variables](../../getting-started/install-self-hosted/environments.md#zhi-shi-ku-pei-zhi);
<!-- TODO: change link -->

Differences in supported file formats for parsing:

| DIFY ETL | Unstructured ETL |
| ---------------------------------------------- | ------------------------------------------------------------------------ |
| txt, markdown, md, pdf, html, htm, xlsx, xls, docx, csv | txt, markdown, md, pdf, html, htm, xlsx, xls, docx, csv, eml, msg, pptx, ppt, xml, epub |

{% hint style="info" %}
Different ETL solutions may have differences in file extraction effects. For more information on Unstructured ETL’s data processing methods, please refer to the [official documentation](https://docs.unstructured.io/open-source/core-functionality/partitioning).
{% endhint %}

***

### 5 Indexing Methods

You need to choose the **indexing method** for the text to specify the data matching method. The indexing strategy is often related to the retrieval method, and you need to choose the appropriate indexing method according to the scenario.

**High-Quality Mode**: Calls OpenAI's embedding interface for processing, providing higher accuracy during user queries.

**Economy Mode**: Uses keyword indexing, reducing accuracy but not requiring token costs.

**Q\&A Mode (community version only)**: The Q\&A segment mode function differs from the ordinary "Q to P" (question to paragraph) matching mode mentioned above. It uses the "Q to Q" (question to question) matching mode. After the document is segmented, each segment generates a Q\&A matching pair through summarization. When a user asks a question, the system finds the most similar question and returns the corresponding segment as the answer. This method is more precise because it directly matches the user’s question, accurately obtaining the information the user truly needs.

When uploading documents to the knowledge base, the system segments the text so that the user's questions (input) can match the relevant text segments (Q to P), and finally output the result.

> Question text is natural language with complete grammatical structure, not just some keywords in a document retrieval task. Therefore, the Q to Q (question matching question) mode makes semantics and matching clearer and meets the needs of high-frequency and high-similarity question scenarios.

<figure><img src="../../.gitbook/assets/Q&A-pair" alt=""><figcaption><p>Texts summarized into multiple Q&A pairs in Q&A segment mode</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/q2p-and-q2q" alt=""><figcaption><p>Difference between Q to P and Q to Q indexing modes</p></figcaption></figure>

***

### 6 Retrieval Settings

In high-quality indexing mode, Dify offers three retrieval options:

* **Vector Search**, generating query embeddings and searching for the text chunk most similar to its vector representation.
* **Full-Text Search**, indexing all terms in the document, allowing users to search any term and retrieve relevant text chunk containing those terms.
* **Hybrid Search**, executing full-text search and vector searches simultaneously, re-rank to select the best match for the user's query. Configuration of the Rerank model APIis necessary.

The specific configurations for the three retrieval methods are as follows:

#### **Vector Search**

Definition: By generating query embeddings to search the most similar text chunk to the query's vector representation. 
<!-- TODO: needs refinement -->

<figure><img src="../../.gitbook/assets/vector-search.png" alt="" width="563"><figcaption><p>Vector Search Settings</p></figcaption></figure>

TopK: Used to filter the text chunk most similar to the user’s query. The system will dynamically adjust the number of chunks based on the context window size of the selected model. The default value is 3.

Score threshold: Used to set the similarity threshold for filtering text fragments, i.e., only recall text fragments that exceed the set score. The system disables this setting by default, meaning no filtering is applied to the similarity values of the recalled text fragments. The default value is 0.5 when enabled.

Rerank model: After configuring the API key for the Rerank model in the “Model Provider” page, you can

 enable the “Rerank model” in the retrieval settings. The system will semantically rerank the recalled document results after vector retrieval to optimize the ranking results. After setting the Rerank model, TopK and Score threshold settings only take effect in the Rerank step.

#### **Full-Text Search**

Definition: Indexing all terms in the document, allowing users to query any terms and return text fragments containing those terms.

<figure><img src="../../.gitbook/assets/full-text-search.png" alt="" width="563"><figcaption><p>Full-Text Search Settings</p></figcaption></figure>

TopK: Used to filter the text fragments most similar to the user’s query. The system will dynamically adjust the number of fragments based on the context window size of the selected model. The default value is 3.

Rerank model: After configuring the API key for the Rerank model in the “Model Provider” page, you can enable the “Rerank model” in the retrieval settings. The system will semantically rerank the recalled document results after full-text retrieval to optimize the ranking results. After setting the Rerank model, TopK and Score threshold settings only take effect in the Rerank step.

#### **Hybrid Search**

Performs full-text and vector search simultaneously, with an additional reranking step to select the best match for the user’s query from the two types of query results, requiring Rerank model API configuration.

<figure><img src="../../.gitbook/assets/hybrid-search.png" alt="" width="563"><figcaption><p>Hybrid Search Settings</p></figcaption></figure>

TopK: Used to filter the text fragments most similar to the user’s query. The system will dynamically adjust the number of fragments based on the context window size of the selected model. The default value is 3.

Rerank model: After configuring the API key for the Rerank model in the “Model Provider” page, you can enable the “Rerank model” in the retrieval settings. The system will semantically rerank the recalled document results after hybrid retrieval to optimize the ranking results. After setting the Rerank model, TopK and Score threshold settings only take effect in the Rerank step.
# How This Program Works

## How `main()` works
* .env is loaded
* Front end is configured and loaded
* `conversation` and `chat_history` are added as keys to `session_state` dictionary-like object
* side bar is loaded
* User uploads docs via st wizardry
* If user hits 'Process' button, the following functions are called:

## Functions
### `get_pdf_text(pdf_docs)`
#### Purpose
To extract the text from the pdf docs and store it all in a variable.
#### Function
* The `pdf_docs` argument is a pdf document or documents uploaded via streamlit's `file_uploader`. It creates `UploadedFile` (objects?) 
* We create a variable called `text` and assign it an empty string. 
* We run a `for` loop through `pdf_docs`. 
* For every `UploadedFile` it creates a `PdfReader(pdf)` object. The `PdfReader()` object has a `.pages` method. 
* A `for` loop within this `for` loop extracts the text from each page and adds it to the `text` variable created at the beginning. 
* The function returns `text`.

### `get_text_chunks(raw_text)`:
#### Purpose
To break up the text into chunks to be embedded later.
#### Function
* We create a variable called `text_splitter` and we assign it the `CharacterTextSplitter` object from LangChain. 
* We then create a `chunks` variable and assign it the result of splitting the text into the chunks specified by the `CharacterTextSplitter` object This splits the text into individual characters and then 'chunks' them together in whatever amount you choose. The function returns `chunks` which will be thousand-word chunks of text.

### `get_vectorstore(text_chunks)`:
#### Purpose
To embed the text chunks into a vectorstore so as to be searchable later.
#### Function
* We take the chunks of text and create 'embeddings' (numberized text for AI) out of them and store them in a vectorstore. 
* We create a variable called `embeddings` and assign it the `OpenAIEmbeddings()` objects. 
* We then create a variable called `vectorstore` and assign it the chunks of text and the OpenAI embedding object. We're using the `from_texts` method from the `FAISS` library to turn `text_chunks` into embeddings via OpenAI and storing them in the `vectorstore` variable.

### `get_conversation_chain(vectorstore)`
#### Purpose
To create a `ConversationRetrievalChain` to allow for questions and answering within the context of documents. This chain is stored in the `st.session_state.converation` object.
#### Function
* We create a variable called `llm` and assign it the `ChatOpenAI()` object. This is essentially an object that uses OpenAI's chat endpoint
* We create a variable called `memory` and assign it the `ConversationBufferMemory()` Object. We pass two arguments to this object: `memory_key='chat_history'` and `return_messages=True`. `ConversationBufferMemory` is a memory utility in the LangChain package that allows for storing messages in a buffer and extracting them as a string or a list of messages.
* We create a variable called `conversation_chain` and assign it the `ConversationalRetrievalChain()` LangChain chain. This allows conversational question answering using a retriever and a question answering chain. The `.from_llm()` constructor allows easily creating an LLMChain from an existing LLM instance, using the LLM's default prompt when called. It saves having to specify a prompt separately.
* The function returns the `conversation_chain`

### `handle_userinput(user_question)`
#### Purpose
Allows the user to ask questions and receive answers within the context of the conversation and processed pdf's.
#### Function
* We add the user's question to the `conversation` object and store that in a variable called `response`
* We add the `chat_history` within the `conversation` object to the `st.session_state.chat_history` object.
* We use a `for` loop with the `enumerate` function to iterate through `st.session_state.chat_history`. For the list item at the 0 index we take the `message.content` and write it to the `user_template`. For the item at the 1 index we write it to the `bot_template`.


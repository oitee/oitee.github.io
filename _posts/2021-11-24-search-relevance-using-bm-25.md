---
layout: post
title: "Search Relevance Using BM25 Algorithm"
tags: project
image: /assets/images/bm25_cover.png
---

Here, I am implementing the BM25 algorithm to retrieve most relevant items out of a set of strings, given a search query. This is the first part of implementing a bare-bones version of [ElasticSearch](https://www.elastic.co/what-is/elasticsearch) which also uses the [BM25 algorithm](https://en.wikipedia.org/wiki/Okapi_BM25) internally.

Before getting into the implementation, following is a quick run down of the algorithm itself.

1. **Documents** are strings, containing words (including repeating words).
2. New documents can be inserted in the store, given a corresponding ID.
3. Documents can be searched using a query string containing one or more words.
4. The search function returns the **most relevant documents first**.
5. Relevance is defined by BM25 to be composed of two factors: **term frequency** and **inverse document frequency**
6. Term frequency is the **number of times a word appears in a document**. For example, if the word 'Clojure' appears 10 times in one document and 2 times in another document, and the query includes the word 'Clojure', the first document will be more relevant.
7. Inverse document frequency **reduces the impact of commonly occuring words**. For example, most documents will contain words like 'this' and 'that'; but few documents will contain the word 'Clojure'. Document frequency tracks the number of documents in which a word appears. So, if the query term contains 'this Clojure', the contribution of the word 'this' to the final score will be less than the word 'Clojure' because, inverse document frequency is defined as 1 / document frequency.

More mathamatically, the scoring function which decides how relevant a document is, given a query, is defined as follows:

<img src="/assets/images/bm25_formula.png" alt="BM25 Equation" border="1px" width="100%"/>

This logic is implemented in [this repository](https://github.com/oitee/bm25). Following is a demo of the final result:


To insert a document, use the following text box:

<textarea id="docInput" rows="4" cols="50">

</textarea>

<button type="button" onclick="insertDoc()">Insert Document</button>

Now to search for the inserted documents, use the following.

<input type="text" id="searchInput">  <button type="button" onclick="searchDoc()">Search</button>


<div id="searchResults"> 

</div>


As an added utility, an IMDB dataset is available to index 1500 movie plots from my GitHub repository, which can be inserted here:

 <button type="button" onclick="populateMovies()">Populate with Movie Plots!</button> 
<div id="populatePrompt">    </div>

 Now, you, for example, you can search with the string "Atlantic Iceberg" and see the movie Titanic as a result🤞.

The code used for this implementation is available here: [https://github.com/oitee/bm25](https://github.com/oitee/bm25). Pull requests are most welcome!

*In a future post, I will compare this implementation with ElasticSearch's results and also provide a REST interface to interact from the external world with this implementation. Stay tuned!*




<script>

/**
 * {integer} K
 * Free parameter for BM25 algorithm
 */
const K = 1.5;

/**
 * {integer} B
 * Free parameter for BM25 algorithm
 */
const B = 0.75;

class Store {
  #documentTF;
  #wordDocFreq;
  #wordCount;
  #docCount;
  #docIndex;

  constructor() {
    /**
     * {Map} #documentTF
     * On a document level, the number of occurences of each word of every document.
     */
    this.#documentTF = new Map();

    /**
     * {Map} #wordDocFreq
     * On a word level, the number of documents each word is present
     */
    this.#wordDocFreq = new Map();

    /**
     * {integer} #wordCount
     * The total number of words in all documents combined, required for calculating average length of documents in words
     */
    this.#wordCount = 0;

    /**
     * {integer} #docCount
     * The total number of documents
     */
    this.#docCount = 0;

    /**
     * {Map} #docIndex
     * All document IDs and their respect documents and the words
     */
    this.#docIndex = new Map();
  }

  /**
   *
   * @param {String} id
   * @param {String} text
   * @returns {null}
   * Inserts a new document in the store
   */
  async insert(id, text) {
    const words = this.#extractWords(text);
    this.#docIndex.set(id, { text: text, words: words });
    const seenWords = new Set();
    const wordsInDoc = new Map();

    words.forEach((word) => {
      //Calculating inverse document frequency
      if (!seenWords.has(word)) {
        if (this.#wordDocFreq.has(word)) {
          this.#wordDocFreq.set(word, this.#wordDocFreq.get(word) + 1);
        } else {
          this.#wordDocFreq.set(word, 1);
        }
      }
      seenWords.add(word);

      // Calculating per-document term frequency
      if (wordsInDoc.has(word)) {
        wordsInDoc.set(word, wordsInDoc.get(word) + 1);
      } else {
        wordsInDoc.set(word, 1);
      }
    });

    this.#documentTF.set(id, wordsInDoc);
    this.#wordCount += words.length;
    this.#docCount++;
  }

  /**
   *
   * @param {string} queryStr
   * @param {integer} limit
   * @returns {object[]}
   * Search the store and return the top matching documents, in order of relevance
   */
  async search(queryStr, limit = 10) {
    const queryTerms = this.#extractWords(queryStr);

    const scoreDocs = await Promise.all(
      this.#docIDs().map(async (id) => {
        const result = await this.#score(id, queryTerms);
        result.text = this.#docIndex.get(id).text.substring(0, 50) + "...";
        return result;
      })
    );
    return scoreDocs
      .filter((doc) => doc.score > 0)
      .sort((a, b) => {
        return b.score - a.score;
      })
      .slice(0, limit);
  }
  
  /**
   * 
   * @returns {integer}
   * Returns the number of documents in the Store
   */
  getDocCount(){
      return this.#docCount;
  }

  //-------------------------------------------------------------
  // PRIVATE METHODS
  //-------------------------------------------------------------

  /**
   *
   * @param {string} str
   * @returns {array}
   * Converts a string into an array of words
   */
  #extractWords(str) {
    return str
      .toLocaleLowerCase()
      .split(/[\W]+/)
      .filter((word) => word.length > 0);
  }

  /**
   *
   * @param {String} docId
   * @param {String[]} queryTerms
   * @returns {Object}
   * Calculate score of current document, for query terms
   * See also: https://en.wikipedia.org/wiki/Okapi_BM25
   */
  async #score(docId, queryTerms) {
    const wordFreq = this.#documentTF.get(docId);

    const docScore = queryTerms.reduce((docScore, query) => {
      const termFrequency = wordFreq.get(query) || 0;
      const wordsInDoc = this.#wordCountInDoc(docId);
      const avgLength = this.#wordCount / this.#docCount;

      docScore +=
        (this.#idf(query) * (termFrequency * (K + 1))) /
        (termFrequency + K * (1 - B + (B * wordsInDoc) / avgLength));
      return docScore;
    }, 0);
    return { score: docScore, id: docId };
  }
  /**
   *
   * @param {string} queryTerm
   * @returns {number}
   * Calculate the inverse document frequency.
   * See also: https://en.wikipedia.org/wiki/Okapi_BM25
   */
  #idf(queryTerm) {
    const documentCount = this.#wordDocFreq.get(queryTerm) || 0;
    return Math.log(
      (this.#wordCount - documentCount + 0.5) / (documentCount + 0.5) + 1
    );
  }

  /**
   *
   * @returns {String[]}
   * Returns all document IDs in the store
   */
  #docIDs() {
    return Array.from(this.#docIndex.keys());
  }

  /**
   *
   * @param {String} id
   * @returns {integer}
   * Returns the count of words in current document
   */
  #wordCountInDoc(id) {
    return this.#docIndex.get(id).words.length;
  }
}
   
const store = new Store();

function insertDoc(){
   const doc = document.getElementById("docInput").value;
   const id = doc.substring(0, 50).replace(/[^0-9a-z]/gi, '_');
   store.insert(id, doc);
   document.getElementById('docInput').value = "";
}

async function searchDoc(){
   const query = document.getElementById("searchInput").value;
   const res = await store.search(query);
   console.log(res);
   let table = ""; 
   table  += `<th style="border: 1px solid black; padding: 5px;"> Sl. No </th>
   <th style="border: 1px solid black; padding: 5px;"> Score </th>
   <th style="border: 1px solid black; padding: 5px;"> ID </th>
   <th style="border: 1px solid black; padding: 5px;"> Text </th>
   `
   let sl = 1;
   for (let {score, text, id} of res){
      table+= `<tr style="border: 1px solid black"> 
      <td style="border: 1px solid black; padding: 5px;"> ${sl}</td>
      <td style="border: 1px solid black; padding: 5px;"> ${Math.round(score * 100) / 100}</td>
      <td style="border: 1px solid black; padding: 5px;"> ${id}</td>
      <td style="border: 1px solid black; padding: 5px;"> ${text}</td>
      </tr>`
      sl++;
   }
   document.getElementById("searchResults").innerHTML = `<table style="width: 100%;  border: 1px solid black; padding: 5px; border-collapse: collapse;"> ${table} </table> <br/><br/>`;
   document.getElementById("searchInput").value = "";
}

async function populateMovies(){
   const rawData = await fetch('https://raw.githubusercontent.com/oitee/bm25/main/dataset/IMDB_movie_details.json')
   .then(res => res.text());

   const documents = rawData.split("\n")
    .map((line) => {
      let lineObj = JSON.parse(line);
      return { text: lineObj["plot_synopsis"], id: lineObj.name };
    });

    await Promise.all(
    documents.map(async ({ text: text, id: id }) => {
      await store.insert(id, text);
    })
  );
  document.getElementById("populatePrompt").innerHTML = "<i> Movies have been populated!</i> <br/>";

}

</script>

